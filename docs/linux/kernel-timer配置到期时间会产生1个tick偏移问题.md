# Kernel-timer配置到期时间会产生一个tick偏移问题分析

## 1 需求

由于用timer_list 可能存在 jiffies 加一的偏差，请帮忙做下面这个实验，把 代码的timer_list 替换成 hrtimer 测下 是有间隔 20ms,代码触发can 发送机制：

```
sudo ip link set can0 up type can bitrate 500000
```

配置 CAN，触发 CAN timer 写循环数据：

```
sudo ip link set can1 up type can bitrate 500000
cansend can1 123#3456 
```

pc 工具统计时间命令：

```
candump -t d can1
```



## 2 问题现象

使用如下代码申请定时器，定时器的执行周期会多增加一个tick，例如下面的代码，timer会间隔2个tick才真正执行：

```
mod_timer(&testtimer, jiffies + 1);
```

timer代码如下：

```c
static void myfunc(unsigned long data)
{
    if(ready_flag == 1){
	printk("can_send1 %lu\n", jiffies);
	struct net_device_stats *stats = &pnet->stats;
    test_quick_priv->write_reg(test_quick_priv, XCAN_ICR_OFFSET, XCAN_IXR_TXOK_MASK);
    test_quick_priv->tx_tail++;
	stats->tx_packets++;

    test_quick_priv->write_reg(test_quick_priv, XCAN_TXFIFO_ID_OFFSET, 0x86400000);
	test_quick_priv->write_reg(test_quick_priv, XCAN_TXFIFO_DLC_OFFSET, 0x80000000);
	test_quick_priv->write_reg(test_quick_priv, XCAN_TXFIFO_DW1_OFFSET, 0x10F0FFFF);
	test_quick_priv->write_reg(test_quick_priv, XCAN_TXFIFO_DW2_OFFSET, 0x10F0FFFF);

    printk("can_send2 %lu\n", jiffies);
    }
    mod_timer(&testtimer, jiffies + 1);
}
```

## 3 问题分析

在《Linux内核设计与实现》11.7.1章节中，有如下描述：

![1537258733790](1537258733790.png)

我们仔细分析下timer的内核代码：

这是相关的数据结构：

```c
#define WHEEL_SIZE	(LVL_SIZE * LVL_DEPTH)

#ifdef CONFIG_NO_HZ_COMMON
# define NR_BASES	2
# define BASE_STD	0
# define BASE_DEF	1
#else
# define NR_BASES	1
# define BASE_STD	0
# define BASE_DEF	0
#endif

struct timer_base {
	raw_spinlock_t		lock;
	struct timer_list	*running_timer;
	unsigned long		clk;
	unsigned long		next_expiry;
	unsigned int		cpu;
	bool			migration_enabled;
	bool			nohz_active;
	bool			is_idle;
	bool			must_forward_clk;
	DECLARE_BITMAP(pending_map, WHEEL_SIZE);
	struct hlist_head	vectors[WHEEL_SIZE];
} ____cacheline_aligned;
```

### 3.1 mod_timer()

在__mod_timer中，如下：

```c
idx = calc_wheel_index(expires, clk);

static int calc_wheel_index(unsigned long expires, unsigned long clk)
{
        unsigned long delta = expires - clk;
        unsigned int idx;

        if (delta < LVL_START(1)) {
        	/*
        	 * 由于expires为jiffies+1，因此进入此
        	 */
                idx = calc_index(expires, 0);
                printk("jiffies %lu, idx %du\n", jiffies, idx);
        } else if (delta < LVL_START(2)) {
                idx = calc_index(expires, 1);
        } else if (delta < LVL_START(3)) {
                idx = calc_index(expires, 2);
        } else if (delta < LVL_START(4)) {
                idx = calc_index(expires, 3);
        } else if (delta < LVL_START(5)) {
                idx = calc_index(expires, 4);
        } else if (delta < LVL_START(6)) {
                idx = calc_index(expires, 5);
        } else if (delta < LVL_START(7)) {
                idx = calc_index(expires, 6);
        } else if (LVL_DEPTH > 8 && delta < LVL_START(8)) {
                idx = calc_index(expires, 7);
        } else if ((long) delta < 0) {
                idx = clk & LVL_MASK;
        } else {
                /*
                 * Force expire obscene large timeouts to expire at the
                 * capacity limit of the wheel.
                 */
                if (expires >= WHEEL_TIMEOUT_CUTOFF)
                        expires = WHEEL_TIMEOUT_MAX;

                idx = calc_index(expires, LVL_DEPTH - 1);
        }
        return idx;
}

static inline unsigned calc_index(unsigned expires, unsigned lvl)
{
	/* expires 为 jiffies + 1，lvl为0
	 * LVL_GRAN: 每个等级的定时器分辨率，用于弥补其低数位的丢失，lvl为0时值为1
	 * LVL_SHIFT: 用于将expires去掉不同分辨率的增幅，用于计算index
	 * 当lvl为0，expires = expires + 1，index = expires & 63
	 * 这应该就是需要下一个jiffies才能匹配到的原因。
	 */
        expires = (expires + LVL_GRAN(lvl)) >> LVL_SHIFT(lvl);
        return LVL_OFFS(lvl) + (expires & LVL_MASK);
}

#define LVL_CLK_SHIFT   3
#define LVL_SHIFT(n)    ((n) * LVL_CLK_SHIFT)
#define LVL_GRAN(n)     (1UL << LVL_SHIFT(n))
```

### 3.2 __run_timers()

```c
/**
 * __run_timers - run all expired timers (if any) on this CPU.
 * @base: the timer vector to be processed.
 */
static inline void __run_timers(struct timer_base *base)
{
	struct hlist_head heads[LVL_DEPTH];
	int levels;

	if (!time_after_eq(jiffies, base->clk))
		return;

	raw_spin_lock_irq(&base->lock);

	while (time_after_eq(jiffies, base->clk)) {

		levels = collect_expired_timers(base, heads);
		base->clk++;

		while (levels--)
			expire_timers(base, heads + levels);
	}
	base->running_timer = NULL;
	raw_spin_unlock_irq(&base->lock);
}
```

collect_expired_timers()

```c
static int collect_expired_timers(struct timer_base *base,
				  struct hlist_head *heads)
{
	/*
	 * NOHZ optimization. After a long idle sleep we need to forward the
	 * base to current jiffies. Avoid a loop by searching the bitfield for
	 * the next expiring timer.
	 */
	if ((long)(jiffies - base->clk) > 2) {
		unsigned long next = __next_timer_interrupt(base);

		/*
		 * If the next timer is ahead of time forward to current
		 * jiffies, otherwise forward to the next expiry time:
		 */
		if (time_after(next, jiffies)) {
			/* The call site will increment clock! */
			base->clk = jiffies - 1;
			return 0;
		}
		base->clk = next;
	}
	return __collect_expired_timers(base, heads);
}
```

```c
static int __collect_expired_timers(struct timer_base *base,
                                    struct hlist_head *heads)
{
        unsigned long clk = base->clk;
        struct hlist_head *vec;
        int i, levels = 0;
        unsigned int idx;

        for (i = 0; i < LVL_DEPTH; i++) {
                idx = (clk & LVL_MASK) + i * LVL_SIZE;

                if (__test_and_clear_bit(idx, base->pending_map)) {
                        vec = base->vectors + idx;
                        hlist_move_list(vec, heads++);
                        levels++;
                }
                /* Is it time to look at the next level? */
                if (clk & LVL_CLK_MASK)
                        break;
                /* Shift clock for the next level granularity */
                clk >>= LVL_CLK_SHIFT;
        }
        return levels;
}

```

index的计算差异：

在__collect_expired_timers中：

```c
/*
 *	LVL_SIZE:	每个Clock等级支持的定时器个数，目前为64个
 *  LVL_MASK:	MASK为63，用于通过clk将Clock等级内的偏移找出来
 */
idx = (clk & LVL_MASK) + i * LVL_SIZE;
```

### 3.3 总结

在timer的call_fn中调用mod_timer()修改定时器下次到期时间，该时间在计算各个timer等级时，将对到期时间+1来计算，这使得run_timer时，根据jiffies来计算index时，只有一下个tick计算出来的index才能匹配到对应的timer，这也印证了《Linux内核设计与实现》里的说法。

## 4 解法

使用高精度定时器解决

```c
#include <linux/kernel.h>
#include <linux/hrtimer.h>
#include <linux/ktime.h>

unsigned long timer_interval_us = 1e6;
static struct hrtimer hr_timer;

static void hrtimer_myfunc(void)
{
    if(ready_flag == 1){

	struct net_device_stats *stats = &pnet->stats;
    test_quick_priv->write_reg(test_quick_priv, XCAN_ICR_OFFSET, XCAN_IXR_TXOK_MASK);
    test_quick_priv->tx_tail++;
	stats->tx_packets++;

    test_quick_priv->write_reg(test_quick_priv, XCAN_TXFIFO_ID_OFFSET, 0x86400000);
	test_quick_priv->write_reg(test_quick_priv, XCAN_TXFIFO_DLC_OFFSET, 0x80000000);
	test_quick_priv->write_reg(test_quick_priv, XCAN_TXFIFO_DW1_OFFSET, 0x10F0FFFF);
	test_quick_priv->write_reg(test_quick_priv, XCAN_TXFIFO_DW2_OFFSET, 0x10F0FFFF);
    }
}

enum hrtimer_restart timer_callback( struct hrtimer *timer_for_restart )
{
	ktime_t currtime , interval;

  	currtime  = ktime_get();
  	interval = ktime_set(0, 20 * timer_interval_us); 
  	hrtimer_forward(timer_for_restart, currtime , interval);
	hrtimer_myfunc();

	return HRTIMER_RESTART;
}

static int hr_timer_init(void) {
	ktime_t ktime;

	ktime = ktime_set(0, 20 * timer_interval_us);
	hrtimer_init(&hr_timer, CLOCK_MONOTONIC, HRTIMER_MODE_REL);
	hr_timer.function = &timer_callback;
	hrtimer_start( &hr_timer, ktime, HRTIMER_MODE_REL );
	return 0;
}

static void timer_exit(void) {
	int ret;
  	ret = hrtimer_cancel( &hr_timer );
  	if (ret) printk("The timer was still in use...\n");
  	printk("HR Timer module uninstalling\n");
}
```

