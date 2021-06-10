---
layout: post
title: "RecyclerView ConcatAdapter"
subtitle: "ConcatAdapter"
author: "XYH"
header-img: ""
header-bg-css: "linear-gradient(to right, #404040, #687a86);"
tags: 
  - RecyclerView
  - Adapter
  - Android
---

[ConcatAdapter](https://developer.android.com/reference/androidx/recyclerview/widget/ConcatAdapter)可以依次呈现多个`RecyclerView#Adapter`的内容，它是由`MergeAdapter`改名而来。

> `MergeAdapter`是在`RecyclerView:1.2.0-alpha02`版本中添加进来的，然后在`RecyclerView:1.2.0-alpha04`改名为[ConcatAdapter](https://developer.android.com/reference/androidx/recyclerview/widget/ConcatAdapter)。

比如常见的给RecyclerView添加Header或者Footer，通过[ConcatAdapter](https://developer.android.com/reference/androidx/recyclerview/widget/ConcatAdapter)可以更加清晰的实现，让代码可维护性更高。

# 实现的效果

![效果图](https://qfxl.oss-cn-shanghai.aliyuncs.com/images/concatAdapter.png)

## 代码实现：

首先依赖RecyclerView。

```xml
implementation 'androidx.recyclerview:recyclerview:1.2.0'
```
### HeaderAdapter。

```xml
<?xml version="1.0" encoding="utf-8"?>
<TextView xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="60dp"
    android:gravity="center"
    android:text="This is header"
    android:textColor="@color/purple_200"
    android:textSize="20sp" />
```

```java
class HeaderAdapter : RecyclerView.Adapter<HeaderAdapter.HeaderViewHolder>() {

    class HeaderViewHolder(view: View) : RecyclerView.ViewHolder(view)
    
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): HeaderViewHolder {
        return HeaderViewHolder(
            LayoutInflater.from(parent.context)
                .inflate(R.layout.recycler_item_header, parent, false)
        )
    }
    
    override fun onBindViewHolder(holder: HeaderViewHolder, position: Int) {}
    
    override fun getItemCount(): Int = 1
}
```

### FooterAdapter

```xml
<?xml version="1.0" encoding="utf-8"?>
<ImageView xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:scaleType="centerInside"
    android:src="@mipmap/ic_launcher"
    android:contentDescription="logo" />
```

```java
class FooterAdapter : RecyclerView.Adapter<FooterAdapter.FooterViewHolder>() {

    class FooterViewHolder(view: View) : RecyclerView.ViewHolder(view)
    
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): FooterViewHolder {
        return FooterViewHolder(
            LayoutInflater.from(parent.context)
                .inflate(R.layout.recycler_item_footer, parent, false)
        )
    }

    override fun onBindViewHolder(holder: FooterViewHolder, position: Int) {}
    
    override fun getItemCount(): Int = 1
}
```

### ListAdapter

```xml
<?xml version="1.0" encoding="utf-8"?>
<TextView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/tv_recycler_item_name"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:textColor="@color/black"
    android:textSize="16sp"
    android:padding="16dp"
    tools:text="Jack" />
```

```java
class NameListAdapter : ListAdapter<String, NameListAdapter.NameViewHolder>(NameDiffCallback) {

    class NameViewHolder(view: View) : RecyclerView.ViewHolder(view) {
        val nameTv: TextView = view.findViewById(R.id.tv_recycler_item_name)
    }
    
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): NameViewHolder {
        return NameViewHolder(
            LayoutInflater.from(parent.context).inflate(R.layout.recycler_item_name, parent, false)
        )
    }

    override fun onBindViewHolder(holder: NameViewHolder, position: Int) {
        holder.nameTv.text = getItem(position)
    }

}

object NameDiffCallback : DiffUtil.ItemCallback<String>() {

    override fun areItemsTheSame(oldItem: String, newItem: String): Boolean {
        return oldItem == newItem
    }

    override fun areContentsTheSame(oldItem: String, newItem: String): Boolean {
        return oldItem == newItem
    }
    
}
```

### 实现

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.recyclerview.widget.RecyclerView android:id="@+id/rv_name_list"
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:layoutManager="androidx.recyclerview.widget.LinearLayoutManager"
    tools:itemCount="3"
    tools:listitem="@layout/recycler_item_name"
    tools:context=".MainActivity"
    xmlns:app="http://schemas.android.com/apk/res-auto" />
```

```java
class MainActivity : AppCompatActivity() {
    private val mHeaderAdapter by lazy {
        HeaderAdapter()
    }
    private val mFooterAdapter by lazy {
        FooterAdapter()
    }
    private val mNameListAdapter by lazy {
        NameListAdapter()
    }
    private val mConcatAdapter by lazy {
        ConcatAdapter(mHeaderAdapter, mNameListAdapter, mFooterAdapter)
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        findViewById<RecyclerView>(R.id.rv_name_list).apply {
            addItemDecoration(DividerItemDecoration(context, DividerItemDecoration.VERTICAL))
            adapter = mConcatAdapter
        }
        mNameListAdapter.submitList(
            listOf(
                "Anna",
                "Bruce",
                "Cindy",
                "Diana",
                "Edison",
                "Ford"
            )
        )
    }
}
```

# ConcatAdapter实现的原理

从对Adapter的理解中，Adapter需要实现的几个重要方法分别为`onCreateViewHolder`、`onBindViewHolder`、`getItemCount`、`getItemViewType`。

源码分析：

```java
ConcatAdapter(mHeaderAdapter, mNameListAdapter, mFooterAdapter)
```
对应源码

```java
public final class ConcatAdapter extends Adapter<ViewHolder> {
  
    @SafeVarargs
    public ConcatAdapter(@NonNull Adapter<? extends ViewHolder>... adapters) {
        this(Config.DEFAULT, adapters);
    }
    
    @Override
    public int getItemCount() {
        return mController.getTotalCount();
    }
    
    @Override
    public int getItemViewType(int position) {
        return mController.getItemViewType(position);
    }    
    
    @NonNull
    @Override
    public ViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        return mController.onCreateViewHolder(parent, viewType);
    }

    @Override
    public void onBindViewHolder(@NonNull ViewHolder holder, int position) {
        mController.onBindViewHolder(holder, position);
    }
}
```

先分析构造函数，构造函数最终调用的函数为

```java
    public ConcatAdapter(
            @NonNull Config config,
            @NonNull List<? extends Adapter<? extends ViewHolder>> adapters) {
        mController = new ConcatAdapterController(this, config);
        for (Adapter<? extends ViewHolder> adapter : adapters) {
            addAdapter(adapter);
        }
        // go through super as we override it to be no-op
        super.setHasStableIds(mController.hasStableIds());
    }
    
    public boolean addAdapter(@NonNull Adapter<? extends ViewHolder> adapter) {
        return mController.addAdapter((Adapter<ViewHolder>) adapter);
    }
```
种种迹象表明，`ConcatAdapterController`才是幕后操作手，因为`ConcatAdapterController`才是负责所有Adapter的逻辑处理，核心代码所在。

走进`ConcatAdapterController`，列出上文调用过的方法。

```java
/**
 * All logic for the {@link ConcatAdapter} is here so that we can clearly see a separation
 * between an adapter implementation and merging logic. 这个注释写的很清楚了
 */
class ConcatAdapterController implements NestedAdapterWrapper.Callback{
    /**
     * return true if added, false otherwise.
     *
     * @see ConcatAdapter#addAdapter(Adapter)
     */
    boolean addAdapter(Adapter<ViewHolder> adapter) {
        return addAdapter(mWrappers.size(), adapter);
    }

    /**
     * return true if added, false otherwise.
     * throws exception if index is out of bounds
     *
     * @see ConcatAdapter#addAdapter(int, Adapter)
     */
    boolean addAdapter(int index, Adapter<ViewHolder> adapter) {
        if (index < 0 || index > mWrappers.size()) {
            throw new IndexOutOfBoundsException("Index must be between 0 and "
                    + mWrappers.size() + ". Given:" + index);
        }
        if (hasStableIds()) {
            Preconditions.checkArgument(adapter.hasStableIds(),
                    "All sub adapters must have stable ids when stable id mode "
                    + "is ISOLATED_STABLE_IDS or SHARED_STABLE_IDS");
        } else {
            if (adapter.hasStableIds()) {
                Log.w(ConcatAdapter.TAG, "Stable ids in the adapter will be ignored as the"
                        + " ConcatAdapter is configured not to have stable ids");
            }
        }
        NestedAdapterWrapper existing = findWrapperFor(adapter);
        if (existing != null) {
            return false;
        }
        NestedAdapterWrapper wrapper = new NestedAdapterWrapper(adapter, this,
                mViewTypeStorage, mStableIdStorage.createStableIdLookup());
        mWrappers.add(index, wrapper);
        // notify attach for all recyclerview
        for (WeakReference<RecyclerView> reference : mAttachedRecyclerViews) {
            RecyclerView recyclerView = reference.get();
            if (recyclerView != null) {
                adapter.onAttachedToRecyclerView(recyclerView);
            }
        }
        // new items, notify add for them
        if (wrapper.getCachedItemCount() > 0) {
            mConcatAdapter.notifyItemRangeInserted(
                    countItemsBefore(wrapper),
                    wrapper.getCachedItemCount()
            );
        }
        // reset state restoration strategy
        calculateAndUpdateStateRestorationPolicy();
        return true;
    }  
    
    public int getTotalCount() {
        // should we cache this as well ?
        int total = 0;
        for (NestedAdapterWrapper wrapper : mWrappers) {
            total += wrapper.getCachedItemCount();
        }
        return total;
    } 
    
    public int getItemViewType(int globalPosition) {
        WrapperAndLocalPosition wrapperAndPos = findWrapperAndLocalPosition(globalPosition);
        int itemViewType = wrapperAndPos.mWrapper.getItemViewType(wrapperAndPos.mLocalPosition);
        releaseWrapperAndLocalPosition(wrapperAndPos);
        return itemViewType;
    } 
    
    public ViewHolder onCreateViewHolder(ViewGroup parent, int globalViewType) {
        NestedAdapterWrapper wrapper = mViewTypeStorage.getWrapperForGlobalType(globalViewType);
        return wrapper.onCreateViewHolder(parent, globalViewType);
    }  

    public void onBindViewHolder(ViewHolder holder, int globalPosition) {
        WrapperAndLocalPosition wrapperAndPos = findWrapperAndLocalPosition(globalPosition);
        mBinderLookup.put(holder, wrapperAndPos.mWrapper);
        wrapperAndPos.mWrapper.onBindViewHolder(holder, wrapperAndPos.mLocalPosition);
        releaseWrapperAndLocalPosition(wrapperAndPos);
    }    
}
```

看看`ConcatAdapterController#addAdapter(int index, Adapter<ViewHolder> adapter)`。

```java
    boolean addAdapter(int index, Adapter<ViewHolder> adapter) {
        if (index < 0 || index > mWrappers.size()) {
            throw new IndexOutOfBoundsException("Index must be between 0 and "
                    + mWrappers.size() + ". Given:" + index);
        }
        //判断是否有stableIds，默认配置是false
        if (hasStableIds()) {
            Preconditions.checkArgument(adapter.hasStableIds(),
                    "All sub adapters must have stable ids when stable id mode "
                    + "is ISOLATED_STABLE_IDS or SHARED_STABLE_IDS");
        } else {
            if (adapter.hasStableIds()) {
                Log.w(ConcatAdapter.TAG, "Stable ids in the adapter will be ignored as the"
                        + " ConcatAdapter is configured not to have stable ids");
            }
        }
        //先从缓存中找该adapter，判断是否添加过，如果已添加就return false
        //NestedAdapterWrapper是Adapter的包装类，看下文。
        NestedAdapterWrapper existing = findWrapperFor(adapter);
        if (existing != null) {
            return false;
        }
        //新建Adapter的包装类
        NestedAdapterWrapper wrapper = new NestedAdapterWrapper(adapter, this,
                mViewTypeStorage, mStableIdStorage.createStableIdLookup());
        //缓存该Adapter的包装类        
        mWrappers.add(index, wrapper);
        // 通知所有绑定了ConcatAdapter的recyclerView，该Adapter绑定了。
        // notify attach for all recyclerview
        for (WeakReference<RecyclerView> reference : mAttachedRecyclerViews) {
            RecyclerView recyclerView = reference.get();
            if (recyclerView != null) {
                adapter.onAttachedToRecyclerView(recyclerView);
            }
        }
        // new items, notify add for them
        //新添加了Adapter，通知ConcatAdapter
        //有新items添加了。
        if (wrapper.getCachedItemCount() > 0) {
            mConcatAdapter.notifyItemRangeInserted(
                    countItemsBefore(wrapper),
                    wrapper.getCachedItemCount()
            );
        }
        // reset state restoration strategy
        // 重置Adapter数据恢复的策略，如果任
        // 意一个adapter的策略是PREVENT那
        // ConncatAdapter则就是PREVENT。        
        calculateAndUpdateStateRestorationPolicy();
        return true;
    }
    
    public boolean hasStableIds() {
        return mStableIdMode != NO_STABLE_IDS;
    }
    
    @Nullable
    private NestedAdapterWrapper findWrapperFor(Adapter<ViewHolder> adapter) {
        final int index = indexOfWrapper(adapter);
        if (index == -1) {
            return null;
        }
        return mWrappers.get(index);
    }
    
    private void calculateAndUpdateStateRestorationPolicy() {
        StateRestorationPolicy newPolicy = computeStateRestorationPolicy();
        if (newPolicy != mConcatAdapter.getStateRestorationPolicy()) {
            mConcatAdapter.internalSetStateRestorationPolicy(newPolicy);
        }
    }  
    
    private StateRestorationPolicy computeStateRestorationPolicy() {
        for (NestedAdapterWrapper wrapper : mWrappers) {
            StateRestorationPolicy strategy =
                    wrapper.adapter.getStateRestorationPolicy();
            if (strategy == PREVENT) {
                // one adapter can block all
                return PREVENT;
            } else if (strategy == PREVENT_WHEN_EMPTY && wrapper.getCachedItemCount() == 0) {
                // an adapter wants to allow w/ size but we need to make sure there is no prevent
                return PREVENT;
            }
        }
        return ALLOW;
    }    
```

`ConcatAdapter#internalSetStateRestorationPolicy(@NonNull StateRestorationPolicy strategy)`

```java
    /**
     * Internal method called by the ConcatAdapterController.
     */
    void internalSetStateRestorationPolicy(@NonNull StateRestorationPolicy strategy) {
        super.setStateRestorationPolicy(strategy);
    }
```

**NestedAdapterWrapper是什么？**

`NestedAdapterWrapper`是ConcatAdapter里所有Adapter的包装，`ConcatAdapterController`很多操作都要该类的参与。

```java
/**
 * Wrapper for each adapter in {@link ConcatAdapter}.
 */
class NestedAdapterWrapper {
    @NonNull
    private final ViewTypeStorage.ViewTypeLookup mViewTypeLookup;
    @NonNull
    private final StableIdStorage.StableIdLookup mStableIdLookup;
    public final Adapter<ViewHolder> adapter;
    @SuppressWarnings("WeakerAccess")
    final Callback mCallback;
    // we cache this value so that we can know the previous size when change happens
    // this is also important as getting real size while an adapter is dispatching possibly a
    // a chain of events might create inconsistencies (as it happens in DiffUtil).
    // Instead, we always calculate this value based on notify events.
    @SuppressWarnings("WeakerAccess")
    int mCachedItemCount;

    private RecyclerView.AdapterDataObserver mAdapterObserver =
            new RecyclerView.AdapterDataObserver() {
                @Override
                public void onChanged() {
                    mCachedItemCount = adapter.getItemCount();
                    mCallback.onChanged(NestedAdapterWrapper.this);
                }

                @Override
                public void onItemRangeChanged(int positionStart, int itemCount) {
                    mCallback.onItemRangeChanged(
                            NestedAdapterWrapper.this,
                            positionStart,
                            itemCount,
                            null
                    );
                }

                @Override
                public void onItemRangeChanged(int positionStart, int itemCount,
                        @Nullable Object payload) {
                    mCallback.onItemRangeChanged(
                            NestedAdapterWrapper.this,
                            positionStart,
                            itemCount,
                            payload
                    );
                }

                @Override
                public void onItemRangeInserted(int positionStart, int itemCount) {
                    mCachedItemCount += itemCount;
                    mCallback.onItemRangeInserted(
                            NestedAdapterWrapper.this,
                            positionStart,
                            itemCount);
                    if (mCachedItemCount > 0
                            && adapter.getStateRestorationPolicy() == PREVENT_WHEN_EMPTY) {
                        mCallback.onStateRestorationPolicyChanged(NestedAdapterWrapper.this);
                    }
                }

                @Override
                public void onItemRangeRemoved(int positionStart, int itemCount) {
                    mCachedItemCount -= itemCount;
                    mCallback.onItemRangeRemoved(
                            NestedAdapterWrapper.this,
                            positionStart,
                            itemCount
                    );
                    if (mCachedItemCount < 1
                            && adapter.getStateRestorationPolicy() == PREVENT_WHEN_EMPTY) {
                        mCallback.onStateRestorationPolicyChanged(NestedAdapterWrapper.this);
                    }
                }

                @Override
                public void onItemRangeMoved(int fromPosition, int toPosition, int itemCount) {
                    Preconditions.checkArgument(itemCount == 1,
                            "moving more than 1 item is not supported in RecyclerView");
                    mCallback.onItemRangeMoved(
                            NestedAdapterWrapper.this,
                            fromPosition,
                            toPosition
                    );
                }

                @Override
                public void onStateRestorationPolicyChanged() {
                    mCallback.onStateRestorationPolicyChanged(
                            NestedAdapterWrapper.this
                    );
                }
            };

    NestedAdapterWrapper(
            Adapter<ViewHolder> adapter,
            final Callback callback,
            ViewTypeStorage viewTypeStorage,
            StableIdStorage.StableIdLookup stableIdLookup) {
        this.adapter = adapter;
        mCallback = callback;
        mViewTypeLookup = viewTypeStorage.createViewTypeWrapper(this);
        mStableIdLookup = stableIdLookup;
        mCachedItemCount = this.adapter.getItemCount();
        this.adapter.registerAdapterDataObserver(mAdapterObserver);
    }


    void dispose() {
        adapter.unregisterAdapterDataObserver(mAdapterObserver);
        mViewTypeLookup.dispose();
    }

    int getCachedItemCount() {
        return mCachedItemCount;
    }

    int getItemViewType(int localPosition) {
        return mViewTypeLookup.localToGlobal(adapter.getItemViewType(localPosition));
    }

    ViewHolder onCreateViewHolder(
            ViewGroup parent,
            int globalViewType) {
        int localType = mViewTypeLookup.globalToLocal(globalViewType);
        return adapter.onCreateViewHolder(parent, localType);
    }

    void onBindViewHolder(ViewHolder viewHolder, int localPosition) {
        adapter.bindViewHolder(viewHolder, localPosition);
    }

    public long getItemId(int localPosition) {
        long localItemId = adapter.getItemId(localPosition);
        return mStableIdLookup.localToGlobal(localItemId);
    }

    interface Callback {
        void onChanged(@NonNull NestedAdapterWrapper wrapper);

        void onItemRangeChanged(
                @NonNull NestedAdapterWrapper nestedAdapterWrapper,
                int positionStart,
                int itemCount
        );

        void onItemRangeChanged(
                @NonNull NestedAdapterWrapper nestedAdapterWrapper,
                int positionStart,
                int itemCount,
                @Nullable Object payload
        );

        void onItemRangeInserted(
                @NonNull NestedAdapterWrapper nestedAdapterWrapper,
                int positionStart,
                int itemCount);

        void onItemRangeRemoved(
                @NonNull NestedAdapterWrapper nestedAdapterWrapper,
                int positionStart,
                int itemCount
        );

        void onItemRangeMoved(
                @NonNull NestedAdapterWrapper nestedAdapterWrapper,
                int fromPosition,
                int toPosition
        );

        void onStateRestorationPolicyChanged(NestedAdapterWrapper nestedAdapterWrapper);
    }
}
```
有了`NestedAdapterWrapper`分析其他几个方法就比较简单了。

**返回数量如何确定？**

在上文中`ConcatAdapter`返回的`itemCount`是️`ConcatAdapterController#getTotalCount`确定的。

```java
public int getTotalCount() {
    // should we cache this as well ?
    int total = 0;
    for (NestedAdapterWrapper wrapper : mWrappers) {
        total += wrapper.getCachedItemCount();
    }
    return total;
}
```
再看`NestedAdapterWrapper#getCachedItemCount`。

```java
int getCachedItemCount() {
    return mCachedItemCount;
}
```
`mCachedItemCount`初始值是`NestedAdapterWrapper`包装Adapter中的`itemCount`，它会随着其Adapter的变化而产生变化，具体对应的代码摘要。

```java
    NestedAdapterWrapper(
            Adapter<ViewHolder> adapter,
            final Callback callback,
            ViewTypeStorage viewTypeStorage,
            StableIdStorage.StableIdLookup stableIdLookup) {
        this.adapter = adapter;
        xxx
        mCachedItemCount = this.adapter.getItemCount();
        this.adapter.registerAdapterDataObserver(mAdapterObserver);
    }    
    
    private RecyclerView.AdapterDataObserver mAdapterObserver =
            new RecyclerView.AdapterDataObserver() {
                @Override
                public void onChanged() {
                    mCachedItemCount = adapter.getItemCount();
                    xxx
                }   
                @Override
                public void onItemRangeChanged(int positionStart, int itemCount) {
                    xxx
                }

                @Override
                public void onItemRangeChanged(int positionStart, int itemCount,
                        @Nullable Object payload) {
                    xxx
                }

                @Override
                public void onItemRangeInserted(int positionStart, int itemCount) {
                    mCachedItemCount += itemCount;
                    xxx
                }

                @Override
                public void onItemRangeRemoved(int positionStart, int itemCount) {
                    mCachedItemCount -= itemCount;
                    xxx
                }

                @Override
                public void onItemRangeMoved(int fromPosition, int toPosition, int itemCount) {
                    xxx
                }

                @Override
                public void onStateRestorationPolicyChanged() {
                    xxx
                }
            };
```

**ViewType如何定义？**

```java
    public int getItemViewType(int globalPosition) {
        WrapperAndLocalPosition wrapperAndPos = findWrapperAndLocalPosition(globalPosition);
        //关键代码所在
        int itemViewType = wrapperAndPos.mWrapper.getItemViewType(wrapperAndPos.mLocalPosition);
        releaseWrapperAndLocalPosition(wrapperAndPos);
        return itemViewType;
    }

    /**
     * Always call {@link #releaseWrapperAndLocalPosition(WrapperAndLocalPosition)} when you are
     * done with it
     */
    @NonNull
    private WrapperAndLocalPosition findWrapperAndLocalPosition(
            int globalPosition
    ) {
        WrapperAndLocalPosition result;
        //mReusableHolder是一个全局复用的WrapperAndLocalPosition
        if (mReusableHolder.mInUse) {
            result = new WrapperAndLocalPosition();
        } else {
            mReusableHolder.mInUse = true;
            result = mReusableHolder;
        }
        int localPosition = globalPosition;
        for (NestedAdapterWrapper wrapper : mWrappers) {
            //找到对应位置的AdapterWrapper并将其赋值给WrapperAndLocalPosition
            if (wrapper.getCachedItemCount() > localPosition) {
                result.mWrapper = wrapper;
                result.mLocalPosition = localPosition;
                break;
            }
            localPosition -= wrapper.getCachedItemCount();
        }
        if (result.mWrapper == null) {
            throw new IllegalArgumentException("Cannot find wrapper for " + globalPosition);
        }
        return result;
    }
    
    private void releaseWrapperAndLocalPosition(WrapperAndLocalPosition wrapperAndLocalPosition) {
        wrapperAndLocalPosition.mInUse = false;
        wrapperAndLocalPosition.mWrapper = null;
        wrapperAndLocalPosition.mLocalPosition = -1;
        mReusableHolder = wrapperAndLocalPosition;
    }
    /**
     * Helper class to hold onto wrapper and local position without allocating objects as this is
     * a very common call.
     */
    static class WrapperAndLocalPosition {
        NestedAdapterWrapper mWrapper;
        int mLocalPosition;
        boolean mInUse;
    }    
```
在上文中的
```java
    public int getItemViewType(int globalPosition) {
        WrapperAndLocalPosition wrapperAndPos = findWrapperAndLocalPosition(globalPosition);
        //关键代码所在
        int itemViewType = wrapperAndPos.mWrapper.getItemViewType(wrapperAndPos.mLocalPosition);
        releaseWrapperAndLocalPosition(wrapperAndPos);
        return itemViewType;
    }
```
有这么一句代码`wrapperAndPos.mWrapper.getItemViewType(wrapperAndPos.mLocalPosition);`，分析一下：
因为`WrapperAndLocalPosition wrapperAndPos = findWrapperAndLocalPosition(globalPosition);`这一句已经给`WrapperAndLocalPosition`赋值了`NestedAdapterWrapper`。

在`NestedAdapterWrapper`种：
```java
class NestedAdapterWrapper {

    @NonNull
    private final ViewTypeStorage.ViewTypeLookup mViewTypeLookup;
    
    int getItemViewType(int localPosition) {
        return mViewTypeLookup.localToGlobal(adapter.getItemViewType(localPosition));
    }
}    
```

`ViewTypeLookup`是个啥？

```java
    /**
     * Api given to {@link NestedAdapterWrapper}s.
     */
    interface ViewTypeLookup {
        int localToGlobal(int localType);

        int globalToLocal(int globalType);

        void dispose();
    }
```

该接口有两个实现，分别为`SharedIdRangeViewTypeStorage#WrapperViewTypeLookup`和`IsolatedViewTypeStorage#IsolatedViewTypeStorage`。
这里需要看下默认使用的是哪个，在

```java
class NestedAdapterWrapper {

    @NonNull
    private final ViewTypeStorage.ViewTypeLookup mViewTypeLookup;
    
    NestedAdapterWrapper(
            Adapter<ViewHolder> adapter,
            final Callback callback,
            ViewTypeStorage viewTypeStorage,
            StableIdStorage.StableIdLookup stableIdLookup) {
        this.adapter = adapter;
        mCallback = callback;
        mViewTypeLookup = viewTypeStorage.createViewTypeWrapper(this);
        mStableIdLookup = stableIdLookup;
        mCachedItemCount = this.adapter.getItemCount();
        this.adapter.registerAdapterDataObserver(mAdapterObserver);
    }    
}
```
这个在`ConcatAdapterController#addAdapter`种初始化`NestedAdapterWrapper`查看用了哪个`ViewTypeStorage`从而决定了`ViewTypeLookup`。

```java
class ConcatAdapterController implements NestedAdapterWrapper.Callback {

    /**
     * Holds the mapping from the view type to the adapter which reported that type.
     */
    private final ViewTypeStorage mViewTypeStorage;    
    ConcatAdapterController(
            ConcatAdapter concatAdapter,
            ConcatAdapter.Config config) {
        mConcatAdapter = concatAdapter;

        // setup view type handling
        if (config.isolateViewTypes) {
            mViewTypeStorage = new ViewTypeStorage.IsolatedViewTypeStorage();
        } else {
            mViewTypeStorage = new ViewTypeStorage.SharedIdRangeViewTypeStorage();
        }

        // setup stable id handling
        mStableIdMode = config.stableIdMode;
        if (config.stableIdMode == NO_STABLE_IDS) {
            mStableIdStorage = new StableIdStorage.NoStableIdStorage();
        } else if (config.stableIdMode == ISOLATED_STABLE_IDS) {
            mStableIdStorage = new StableIdStorage.IsolatedStableIdStorage();
        } else if (config.stableIdMode == SHARED_STABLE_IDS) {
            mStableIdStorage = new StableIdStorage.SharedPoolStableIdStorage();
        } else {
            throw new IllegalArgumentException("unknown stable id mode");
        }
    }
    
    boolean addAdapter(int index, Adapter<ViewHolder> adapter) {
        xxx
        NestedAdapterWrapper wrapper = new NestedAdapterWrapper(adapter, this,
                mViewTypeStorage, mStableIdStorage.createStableIdLookup());
        xxx
        return true;
    }    
    
}
```
决定使用哪个`ViewTypeStorage`要根据对应`Config`的`isolateViewTypes`来决定。

在最开始`ConcatAdapter`中初始化`ConcatAdapterController`的时候用的是`Config.DEFAULT`。

```java
public final class ConcatAdapter extends Adapter<ViewHolder> {
    xxx

    @SafeVarargs
    public ConcatAdapter(@NonNull Adapter<? extends ViewHolder>... adapters) {
        this(Config.DEFAULT, adapters);
    }

    @SafeVarargs
    public ConcatAdapter(
            @NonNull Config config,
            @NonNull Adapter<? extends ViewHolder>... adapters) {
        this(config, Arrays.asList(adapters));
    }

    public ConcatAdapter(@NonNull List<? extends Adapter<? extends ViewHolder>> adapters) {
        this(Config.DEFAULT, adapters);
    }

    public ConcatAdapter(
            @NonNull Config config,
            @NonNull List<? extends Adapter<? extends ViewHolder>> adapters) {
        mController = new ConcatAdapterController(this, config);
        for (Adapter<? extends ViewHolder> adapter : adapters) {
            addAdapter(adapter);
        }
        // go through super as we override it to be no-op
        super.setHasStableIds(mController.hasStableIds());
    }    
    xxx
    public static final class Config {
        public final boolean isolateViewTypes;
        
        @NonNull
        public final StableIdMode stableIdMode;
     
         @NonNull
        public static final Config DEFAULT = new Config(true, NO_STABLE_IDS);
        
        Config(boolean isolateViewTypes, @NonNull StableIdMode stableIdMode) {
            this.isolateViewTypes = isolateViewTypes;
            this.stableIdMode = stableIdMode;
        }
    }    
}
```
分析得之，`ViewTypeStorage`使用的是`ViewTypeStorage.IsolatedViewTypeStorage()`。

回到`NestedAdapterWrapper`的构造函数种。
```java
    NestedAdapterWrapper(
            Adapter<ViewHolder> adapter,
            final Callback callback,
            ViewTypeStorage viewTypeStorage,
            StableIdStorage.StableIdLookup stableIdLookup) {
        this.adapter = adapter;
        mCallback = callback;
        mViewTypeLookup = viewTypeStorage.createViewTypeWrapper(this);
        mStableIdLookup = stableIdLookup;
        mCachedItemCount = this.adapter.getItemCount();
        this.adapter.registerAdapterDataObserver(mAdapterObserver);
    }
```
`mViewTypeLookup = viewTypeStorage.createViewTypeWrapper(this);`调用的是`IsolatedViewTypeStorage#createViewTypeWrapper`。


```java
    class IsolatedViewTypeStorage implements ViewTypeStorage {
        SparseArray<NestedAdapterWrapper> mGlobalTypeToWrapper = new SparseArray<>();

        int mNextViewType = 0;

        int obtainViewType(NestedAdapterWrapper wrapper) {
            //很关键，这里存的key是ViewType，value是对应的AdapterWrapper。
            int nextId = mNextViewType++;
            mGlobalTypeToWrapper.put(nextId, wrapper);
            return nextId;
        }
        //根据ViewType返回对应的AdapterWrapper，obtainViewType有存。
        @NonNull
        @Override
        public NestedAdapterWrapper getWrapperForGlobalType(int globalViewType) {
            NestedAdapterWrapper wrapper = mGlobalTypeToWrapper.get(
                    globalViewType);
            if (wrapper == null) {
                throw new IllegalArgumentException("Cannot find the wrapper for global"
                        + " view type " + globalViewType);
            }
            return wrapper;
        }

        @Override
        @NonNull
        public ViewTypeLookup createViewTypeWrapper(
                @NonNull NestedAdapterWrapper wrapper) {
            return new WrapperViewTypeLookup(wrapper);
        }

        void removeWrapper(@NonNull NestedAdapterWrapper wrapper) {
            for (int i = mGlobalTypeToWrapper.size() - 1; i >= 0; i--) {
                NestedAdapterWrapper existingWrapper = mGlobalTypeToWrapper.valueAt(i);
                if (existingWrapper == wrapper) {
                    mGlobalTypeToWrapper.removeAt(i);
                }
            }
        }

        class WrapperViewTypeLookup implements ViewTypeLookup {
            private SparseIntArray mLocalToGlobalMapping = new SparseIntArray(1);
            private SparseIntArray mGlobalToLocalMapping = new SparseIntArray(1);
            final NestedAdapterWrapper mWrapper;

            WrapperViewTypeLookup(NestedAdapterWrapper wrapper) {
                mWrapper = wrapper;
            }
            //可以理解为localToGlobal是将adapter返回的ViewType转换成ConcatAdapter需要的ViewType。
            @Override
            public int localToGlobal(int localType) {
                //查询是否有过转换记录
                int index = mLocalToGlobalMapping.indexOfKey(localType);
                if (index > -1) {
                    return mLocalToGlobalMapping.valueAt(index);
                }
                // get a new key.
                //返回的就是设置给当前AdapterWrapper的ViewType
                int globalType = obtainViewType(mWrapper);
                mLocalToGlobalMapping.put(localType, globalType);
                mGlobalToLocalMapping.put(globalType, localType);
                return globalType;
            }
            //这个就是将转换的ViewType还原回去
            @Override
            public int globalToLocal(int globalType) {
                int index = mGlobalToLocalMapping.indexOfKey(globalType);
                if (index < 0) {
                    throw new IllegalStateException("requested global type " + globalType + " does"
                            + " not belong to the adapter:" + mWrapper.adapter);
                }
                return mGlobalToLocalMapping.valueAt(index);
            }

            @Override
            public void dispose() {
                removeWrapper(mWrapper);
            }
        }
    }
```

至此我们可以知道`ConcatAdapter`会将其包含的`Adapter`返回的`ViewType`转换成其需要的`ViewType`，并且可以将这个转换之后的`ViewType`可以轻松还原成默认的`ViewType`。


**ViewHolder如何创建？**

在`ConcatAdapterController`中

```java
class ConcatAdapterController implements NestedAdapterWrapper.Callback {
    public ViewHolder onCreateViewHolder(ViewGroup parent, int globalViewType) {
        //根据ViewType找到对应的AdapterWrapper，上面有代码提到。
        NestedAdapterWrapper wrapper = mViewTypeStorage.getWrapperForGlobalType(globalViewType);
        //调用Wrapper的onCreateViewHolder
        return wrapper.onCreateViewHolder(parent, globalViewType);
    }    
}
```

在`NestedWrapperAdapter`中

```java
class NestedAdapterWrapper {
    ViewHolder onCreateViewHolder(
            ViewGroup parent,
            int globalViewType) {
        //将ConcatAdapter中的ViewType还原成各个Adapter自己的ViewType
        int localType = mViewTypeLookup.globalToLocal(globalViewType);
        return adapter.onCreateViewHolder(parent, localType);
    }
}    
```

**ViewHolder如何绑定？**

在`ConcatAdapterController`中：

```java
class ConcatAdapterController implements NestedAdapterWrapper.Callback {
    /**
     * Keeps the information about which ViewHolder is bound by which adapter.
     * It is set in onBind, reset at onRecycle.
     */
    private final IdentityHashMap<ViewHolder, NestedAdapterWrapper>
            mBinderLookup = new IdentityHashMap<>();
            
    public void onBindViewHolder(ViewHolder holder, int globalPosition) {
        WrapperAndLocalPosition wrapperAndPos = findWrapperAndLocalPosition(globalPosition);
        //存储ViewHolder跟NestedAdapterWrapper对应关系，主要作用有在ConcatAdapter的onViewAttachedToWindow、onViewDetachedFromWindow、onFailedToRecycleView方法中能够根据ViewHolder找到对应NestedAdapterWrapper然后通知包含的Adapter。
        mBinderLookup.put(holder, wrapperAndPos.mWrapper);
        //调用NestedAdapterWrapper的onBindViewHolder
        wrapperAndPos.mWrapper.onBindViewHolder(holder, wrapperAndPos.mLocalPosition);
        releaseWrapperAndLocalPosition(wrapperAndPos);
    }
}
```

```java
class NestedAdapterWrapper {
    void onBindViewHolder(ViewHolder viewHolder, int localPosition) {
        adapter.bindViewHolder(viewHolder, localPosition);
    }
}    
```

**Concat的Adapter如果发生变化怎么通知ConcatAdapter？**

首先`ConcatAdapterController`实现了`NestedAdapterWrapper.Callback`接口，点进源码分析：

```java
class ConcatAdapterController implements NestedAdapterWrapper.Callback {
  @Override
    public void onChanged(@NonNull NestedAdapterWrapper wrapper) {
        // 有点意思，我也觉得这里应该局部刷新。
        // TODO should we notify more cleverly, maybe in v2
        mConcatAdapter.notifyDataSetChanged();
        calculateAndUpdateStateRestorationPolicy();
    }

    @Override
    public void onItemRangeChanged(@NonNull NestedAdapterWrapper nestedAdapterWrapper,
            int positionStart, int itemCount) {
        final int offset = countItemsBefore(nestedAdapterWrapper);
        mConcatAdapter.notifyItemRangeChanged(
                positionStart + offset,
                itemCount
        );
    }

    @Override
    public void onItemRangeChanged(@NonNull NestedAdapterWrapper nestedAdapterWrapper,
            int positionStart, int itemCount, @Nullable Object payload) {
        final int offset = countItemsBefore(nestedAdapterWrapper);
        mConcatAdapter.notifyItemRangeChanged(
                positionStart + offset,
                itemCount,
                payload
        );
    }

    @Override
    public void onItemRangeInserted(@NonNull NestedAdapterWrapper nestedAdapterWrapper,
            int positionStart, int itemCount) {
        final int offset = countItemsBefore(nestedAdapterWrapper);
        mConcatAdapter.notifyItemRangeInserted(
                positionStart + offset,
                itemCount
        );
    }

    @Override
    public void onItemRangeRemoved(@NonNull NestedAdapterWrapper nestedAdapterWrapper,
            int positionStart, int itemCount) {
        int offset = countItemsBefore(nestedAdapterWrapper);
        mConcatAdapter.notifyItemRangeRemoved(
                positionStart + offset,
                itemCount
        );
    }

    @Override
    public void onItemRangeMoved(@NonNull NestedAdapterWrapper nestedAdapterWrapper,
            int fromPosition, int toPosition) {
        int offset = countItemsBefore(nestedAdapterWrapper);
        mConcatAdapter.notifyItemMoved(
                fromPosition + offset,
                toPosition + offset
        );
    }    
}
```

```java
class NestedAdapterWrapper {
     private RecyclerView.AdapterDataObserver mAdapterObserver =
            new RecyclerView.AdapterDataObserver() {
                @Override
                public void onChanged() {
                    mCachedItemCount = adapter.getItemCount();
                    mCallback.onChanged(NestedAdapterWrapper.this);
                }

                @Override
                public void onItemRangeChanged(int positionStart, int itemCount) {
                    mCallback.onItemRangeChanged(
                            NestedAdapterWrapper.this,
                            positionStart,
                            itemCount,
                            null
                    );
                }

                @Override
                public void onItemRangeChanged(int positionStart, int itemCount,
                        @Nullable Object payload) {
                    mCallback.onItemRangeChanged(
                            NestedAdapterWrapper.this,
                            positionStart,
                            itemCount,
                            payload
                    );
                }

                @Override
                public void onItemRangeInserted(int positionStart, int itemCount) {
                    mCachedItemCount += itemCount;
                    mCallback.onItemRangeInserted(
                            NestedAdapterWrapper.this,
                            positionStart,
                            itemCount);
                    if (mCachedItemCount > 0
                            && adapter.getStateRestorationPolicy() == PREVENT_WHEN_EMPTY) {
                        mCallback.onStateRestorationPolicyChanged(NestedAdapterWrapper.this);
                    }
                }

                @Override
                public void onItemRangeRemoved(int positionStart, int itemCount) {
                    mCachedItemCount -= itemCount;
                    mCallback.onItemRangeRemoved(
                            NestedAdapterWrapper.this,
                            positionStart,
                            itemCount
                    );
                    if (mCachedItemCount < 1
                            && adapter.getStateRestorationPolicy() == PREVENT_WHEN_EMPTY) {
                        mCallback.onStateRestorationPolicyChanged(NestedAdapterWrapper.this);
                    }
                }

                @Override
                public void onItemRangeMoved(int fromPosition, int toPosition, int itemCount) {
                    Preconditions.checkArgument(itemCount == 1,
                            "moving more than 1 item is not supported in RecyclerView");
                    mCallback.onItemRangeMoved(
                            NestedAdapterWrapper.this,
                            fromPosition,
                            toPosition
                    );
                }

                @Override
                public void onStateRestorationPolicyChanged() {
                    mCallback.onStateRestorationPolicyChanged(
                            NestedAdapterWrapper.this
                    );
                }
            };
            
    NestedAdapterWrapper(
            Adapter<ViewHolder> adapter,
            final Callback callback,
            ViewTypeStorage viewTypeStorage,
            StableIdStorage.StableIdLookup stableIdLookup) {
        this.adapter = adapter;
        xxx
        mCachedItemCount = this.adapter.getItemCount();
        this.adapter.registerAdapterDataObserver(mAdapterObserver);
    }            
}
```

当`ConcatAdapter`包含的`Adapter`发生数据变化的时候，会通过`Callback`接口回调给`ConcatAdapterController`。

# 总结

[ConcatAdapter](https://developer.android.com/reference/androidx/recyclerview/widget/ConcatAdapter)之所以能依次呈现多个`Adapter`的内容，其本质还是通过`Adapter`的不同`ViewType`创建不同的`ViewHolder`关联RecyclerView来实现，所以学好基础很重要。



