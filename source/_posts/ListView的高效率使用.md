title: ListView的高效率使用
tags: 'ListView, 优化'
categories: Android
date: 2014-07-23 12:00:16
---
ListView作为android上最常用的控件之一，其原理与效率十分值得我们去探究，特别是当与Button、ImageView一起时，许多问题便纷至沓来。今天我就说说这老生常谈的话题，也是由此做一个总结。
 
## 一，ListView的原理

在Android中有一个Recycler的构件，滑出屏幕的的Item就会被缓存在里面。但并不是缓存全部，只是会根据item的类型数来缓存相应的个数。举个例子，一般情况下都没有必要去重写ListView的getViewTypeCount())函数，那么type值也就为1，也就是说，Recycler中只会缓存一个item View。如下图（图片来自网络）：
 
 ![](http://zhenghuiy-blog.qiniudn.com/recycler.jpeg)

当Item1滑出屏幕的时候，Recycler构件中就缓存了item1,再往上滑，item2滑出屏幕。这时有趣的事情就发生了，Recycler构件只会缓存一个item，这时毫无争议会缓存Item2，那么item1就被复用，显示为item8（实际上当item1完全滑出界面时就会被显示作Item8了）。

## 二，需要注意的地方

### 1，数据与显示分离

由于ListView的``item view``是复用的，这就要求我们将数据与显示分离，否则将造成显示的结果错乱。比如，当ListView的``Item``中含有单选按钮时，假如只是简单地记录按钮的点击状态，那么当滑动ListView的时候，“已点击”与“未点击”就显示混乱。再举个例子，ListView的``item``中含有图片，选择异步加载。如果做法是当下载完毕后就设置ImageView的显示内容，那么也会造成混乱。

比较明智的做法是自己单独维护一个数据结构来缓存数据，上文的点击状态就时候使用``ArrayList``,异步加载的图片就适合使用``HashMap``。

### 2，数据更新

ListView的数据更新只能在UI主线程中进行，否则会报：
```
“java.lang.IllegalStateException: The content of the adapter has changed but ListView did not receive a notification. Make sure the content of your adapter is not modified from a background thread, but only from the UI thread.”
```
另外还需要注意的是，不能更改原来的数据指向的对象。比如，原来的数据``List`` 指向的是 ``ArrayList1``。在数据更新后，为了方便，我们直接 ``List = ArrayList2``。这样就使``List``指向新的``ArrayList``，也是会出错的。总而言之，唯一的做法就是在原理的ArrayList1中增加或删除数据。

<!--more-->
### 3，使用BitMap需要注意不要OOM

使用ListView的场景里一般含有多条数据，每条数据都有``BitMap``的情况下很容易造成OOM。比较合理的做法是，``Bitmap``要及时回收，同时，要合理地压缩``Bitmap``。

## 三，ListView的优化

### 1，判断item的view是否为空

如果view为空，就不需要去将XML布局文件转化为View,也就是达到使用recycler缓存的目的。

### 2，使用ViewHolder

在Android中，一个界面可以被组织为一棵控件树，View的绘制就是围绕这棵控件树来执行的。而``view.findViewById(int id)``函数的原理是，从该View开始（包含该view）进行树的深度优先遍历，匹配最先找到的id为参数id的view（因此android中的id不要求全局唯一，只要能保证准确获取就行）。所以，``findViewById()``这个函数是十分耗时的。使用一个内部类ViewHolder可以缓存住item的子控件，使其只需要进行N次遍历（N = item中控件数 * 一屏能显示的item个数）。

以下是例子:
```
public class MBaseAdapter extends BaseAdapter{

    List mList;
    
    Context mContext;
    
    ViewHolder mViewHolder;
    
    public MBaseAdapter(Context context,List list) {
    
        mContext = context;
        
        mList = list;
    
    }
    
    @Override
    
    public int getCount() {
    
        return mList.size();
    
    }
    
    @Override
    
    public Object getItem(int position) {
    
        return mList.get(position);
    
    }
    
    @Override
    
    public long getItemId(int position) {
    
    return position;
    
    }
    
    @Override
    
        public View getView(int position, View convertView, ViewGroup arg2) {
        
        if (convertView == null) {
        
        convertView = LayoutInflater.from(mContext).inflate(R.layout.list_item, null);
        
        mViewHolder = new ViewHolder();
        
        mViewHolder.tvHello = (TextView) convertView.findViewById(R.id.list_hello);
        
        convertView.setTag(mViewHolder);
        
        } else {
        
        mViewHolder = (ViewHolder)convertView.getTag();
        
        }
        
        mViewHolder.tvHello.setText(mList.get(position)+" Hello,World!");
        
        return convertView;
        
    }
    
    class ViewHolder {
    
        TextView tvHello;
    
    }

}
```
### 3，在合适的状态显示合适的数据

这里主要是监听滑动状态，当状态为滑动或者惯性滑动时不需要开加载图片，当状态为普通显示时就加载图片。