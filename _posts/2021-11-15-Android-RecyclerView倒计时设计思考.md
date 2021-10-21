---
layout: post
title: "RecyclerView Item倒计时设计的思考"
subtitle: "RecyclerView Countdown"
author: "XYH"
header-img: ""
header-bg-css: "linear-gradient(to right, #404040, #687a86);"
tags: 
  - RecyclerView
  - Android
---

# 前言

RecyclerView Item倒计时这个需求比较常见于商品的秒杀，活动的预告之类，在之前项目中采用的实现方式是ViewHolder持有CountDownTimer，虽然这种方式可以实现Item倒计时，但是给每个ViewHolder设计一个CountDownTimer无疑增加了CPU的开销以及内存的浪费，之后尝试思考能不能使用一个轮询来实现这种效果，一番思考之后想到了解决方案。

思路：
* Adapter持有一个轮询
* Adapter轮询tick的时候更新对应ViewHolder里面的UI
* 更新数值为Item剩余时间减去轮询持续的时间

# 效果

![recycle_timer](https://qfxl.oss-cn-shanghai.aliyuncs.com/images/recycle_timer.gif)

# 代码

准备商品Bean：

```java
data class Goods(val id: Int, val goodsDesc: String, val leftTime: Long)
```

Adapter：

```java
class GoodsAdapter : ListAdapter<Goods, GoodsAdapter.ViewHolder>(Differ) {

    private object Differ : DiffUtil.ItemCallback<Goods>() {
        override fun areItemsTheSame(oldItem: Goods, newItem: Goods): Boolean {
            return oldItem.id == newItem.id
        }

        override fun areContentsTheSame(oldItem: Goods, newItem: Goods): Boolean {
            return oldItem.id == newItem.id
        }
    }

    companion object {
        private const val INTERVAL_WHAT = 0x001
    }

    private var millsCounted = 0L

    private val mObservers by lazy {
        ArrayList<OnTickAble>()
    }

    private val intervalHandler by lazy {
        object : Handler(Looper.getMainLooper()) {
            override fun handleMessage(msg: Message) {
                millsCounted += 1000L
                onTick()
                sendEmptyMessageDelayed(INTERVAL_WHAT, 1000L)
            }
        }
    }

    class ViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView), OnTickAble {
        val goodsDescTv: TextView = itemView.findViewById(R.id.tv_recycler_item_goods_goods_desc)
        val goodsLeftTimeTv: TextView =
            itemView.findViewById(R.id.tv_recycler_item_goods_left_time)

        override fun onTick(mills: Long) {
            val mTag = goodsLeftTimeTv.tag
            if (mTag is Long) {
                val validTime = if (mTag - mills > 0) mTag - mills else 0
                if (validTime > 0) {
                    goodsLeftTimeTv.text = "剩余 ${validTime / 1000L} 秒"
                } else {
                    goodsLeftTimeTv.text = "抢购结束"
                }
            }
        }
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
        return ViewHolder(
            LayoutInflater.from(parent.context).inflate(R.layout.recycler_item_goods, parent, false)
        ).apply {
            mObservers.add(this)
        }
    }

    override fun onBindViewHolder(holder: ViewHolder, position: Int) {
        val item = getItem(position)
        holder.apply {
            val validTime =
                if (item.leftTime - millsCounted > 0) item.leftTime - millsCounted else 0
            goodsLeftTimeTv.tag = item.leftTime
            if (validTime > 0) {
                goodsLeftTimeTv.text = "剩余 ${validTime / 1000L} 秒"
            } else {
                goodsLeftTimeTv.text = "抢购结束"
            }
            goodsDescTv.text = item.goodsDesc
        }
    }

    override fun onAttachedToRecyclerView(recyclerView: RecyclerView) {
        super.onAttachedToRecyclerView(recyclerView)
        millsCounted = 0L
        intervalHandler.sendEmptyMessageDelayed(INTERVAL_WHAT, 1000L)
    }

    override fun onDetachedFromRecyclerView(recyclerView: RecyclerView) {
        super.onDetachedFromRecyclerView(recyclerView)
        intervalHandler.removeMessages(INTERVAL_WHAT)
    }

    private fun onTick() {
        mObservers.forEach {
            it.onTick(millsCounted)
        }
    }

    interface OnTickAble {
        fun onTick(mills: Long)
    }
}
```

recycler_item_goods：

```java
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:paddingStart="16dp"
    android:paddingEnd="16dp">

    <TextView
        android:id="@+id/tv_recycler_item_goods_goods_desc"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:ellipsize="end"
        android:maxLines="1"
        android:textColor="@color/black"
        android:textSize="18sp"
        app:layout_constrainedWidth="true"
        app:layout_constraintBottom_toBottomOf="@id/btn_recycler_item_goods_buy"
        app:layout_constraintEnd_toStartOf="@id/btn_recycler_item_goods_buy"
        app:layout_constraintHorizontal_bias="0"
        app:layout_constraintHorizontal_chainStyle="packed"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        tools:text="This is goods info This is goods info This is goods info" />

    <Button
        android:id="@+id/btn_recycler_item_goods_buy"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginStart="16dp"
        android:text="Buy"
        android:textSize="18sp"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toEndOf="@id/tv_recycler_item_goods_goods_desc"
        app:layout_constraintTop_toTopOf="parent" />

    <androidx.constraintlayout.widget.Barrier
        android:id="@+id/barrier_recycler_item_goods_goods_desc"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        app:barrierDirection="bottom"
        app:constraint_referenced_ids="btn_recycler_item_goods_buy" />

    <TextView
        android:id="@+id/tv_recycler_item_goods_left_time"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="16dp"
        android:layout_marginBottom="8dp"
        android:textSize="16sp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintHorizontal_bias="0"
        app:layout_constraintStart_toStartOf="@id/tv_recycler_item_goods_goods_desc"
        app:layout_constraintTop_toBottomOf="@id/barrier_recycler_item_goods_goods_desc"
        tools:text="188s" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

MainActivity：

```java
class MainActivity : AppCompatActivity() {

    private lateinit var binding: ActivityMainBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        val mockList = listOf(
            Goods(0, "独束大码早秋连衣裙", 10 * 1000L),
            Goods(1, "PLUS2021新款女秋装气质针织连衣裙女长袖小黑裙法式茶歇裙", 15 * 1000L),
            Goods(2, "独束大码撞色减龄海军风连衣裙女秋冬2021年新款胖MM长袖显瘦长裙", 20 * 1000L),
            Goods(3, "2021秋冬新款后背镂空泡泡袖连衣裙|32137B032", 25 * 1000L),
            Goods(4, "吃绿色菜 早秋赫本风西装翻领牛仔连衣裙女秋装中长款收腰复古裙", 30 * 1000L),
            Goods(5, "奥利奥夹心 包臀裙子秋装2021年新款格子吊带裙内搭连衣裙", 35 * 1000L),
            Goods(6, "GUOGE红色连衣裙女2021夏季新款方领法式复古气质泡泡袖订婚裙", 40 * 1000L),
            Goods(7, "赫本风娃娃领小黑裙子2021年春秋新款女装大翻领收腰黑色连衣裙女", 45 * 1000L),
            Goods(8, "脚趾筋弗朗的夜设计感法式连衣裙女微胖MM秋冬宽松显瘦拼接长裙子", 50 * 1000L),
            Goods(9, "2021秋冬新款气质淑女中长款温柔珍珠大领子针织连衣裙女", 55 * 1000L),
            Goods(10, "伊芙丽新款仙女裙蕾丝拼接连衣裙+V领黑白针织衫", 60 * 1000L),
            Goods(11, "长袖初恋连衣裙女早秋温柔风气质设计感荷叶边白色a字长裙", 65 * 1000L),
            Goods(12, "ModaJayde Fish联名秋季新款气质复古印花长款仙女连衣裙", 70 * 1000L),
            Goods(13, "法式长袖连衣裙2021新款秋季显瘦长裙女气质别致设计感小众白裙子", 75 * 1000L),
            Goods(14, "气质温柔风抽褶连衣裙女早秋新款木耳边减龄休闲A字茶歇裙", 80 * 1000L),
            Goods(15, "法式不规则长袖裙子早秋气质时尚新款黑色假两件连衣裙女高端显瘦", 85 * 1000L),
            Goods(16, "泡泡袖宽松版型缩褶连衣裙浅粉色2021秋季新品1027941002", 90 * 1000L)
        )

        binding.root.apply {
            addItemDecoration(
                DividerItemDecoration(
                    context,
                    DividerItemDecoration.VERTICAL
                )
            )
            adapter = GoodsAdapter().apply {
                submitList(mockList)
            }
        }
    }
}
```

activity_main：

```xml
<?xml version="1.0" encoding="utf-8"?>
<cn.xuyonghong.recycletimer.LifecycleRecyclerView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:layoutManager="androidx.recyclerview.widget.LinearLayoutManager"
    tools:context=".MainActivity"
    tools:itemCount="2"
    tools:listitem="@layout/recycler_item_goods" />
```

为了实现Adapter从RecyclerView解绑的时候停止轮询，需要对RecyclerView稍微处理一下。

```java
class LifecycleRecyclerView @JvmOverloads constructor(
    context: Context,
    attrs: AttributeSet? = null,
    defStyle: Int = R.attr.recyclerViewStyle
) : RecyclerView(context, attrs, defStyle), LifecycleObserver {

    init {
        if (context is LifecycleOwner) {
            context.lifecycle.addObserver(this)
        }
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_DESTROY)
    fun onDestroy() {
        adapter = null
    }

    override fun setLayoutManager(layout: LayoutManager?) {
        super.setLayoutManager(layout)
        if (layout is LinearLayoutManager) {
            layout.recycleChildrenOnDetach = true
        }
    }
}
```

对于轮询的最大时间，自适应停止之类的需求都可以在该基础上进行拓展。