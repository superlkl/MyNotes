# ListView

## 最简单的方式

```kotlin
class MainActivity : AppCompatActivity() {
    //1.准备数据
    private val data = listOf("孙悟空","唐三藏","沙和尚","猪八戒","牛魔王","铁扇公主","蜘蛛精","白骨精",
    "二郎神","哮天犬","哪吒","托塔李天王","玉皇大帝","王母娘娘","如来佛祖","四大天王","七仙女","黄牛怪")

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        //2.在布局文件中引用ListView
        setContentView(R.layout.activity_main)
        //3.创建适配器
        val arrayAdapter =
            ArrayAdapter(this, android.R.layout.simple_list_item_1, data)
        //4.通过ListView的id给其设置适配器
        list_view.adapter = arrayAdapter
    }
}
```

- **R.layout.activity_main布局文件**

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".MainActivity">

<ListView
    android:id="@+id/list_view"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
</ListView>

</LinearLayout>
```

- 点击 list_view.adapter 进入查看，实际上是调用了setAdapter()的方法

```java
@Override
public void setAdapter(ListAdapter adapter) {
    if (mAdapter != null && mDataSetObserver != null) {
        mAdapter.unregisterDataSetObserver(mDataSetObserver);
    }
    //...
}
```

- **ArrayAdapter需要传入三个参数**
  1. 上下文对象，一般传入this
  2. 为ListView每个item同一适配需要的布局文件id
  3. 数据

```java
public ArrayAdapter(@NonNull Context context, @LayoutRes int resource,
        @NonNull List<T> objects) {
    this(context, resource, 0, objects);
}
```

- android.R.layout.simple_list_item_1是系统自带的一个布局文件
  - 仅仅就是放置了一个TextView控件

```xml
<?xml version="1.0" encoding="utf-8"?>
<TextView xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@android:id/text1"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:textAppearance="?android:attr/textAppearanceListItemSmall"
    android:gravity="center_vertical"
    android:paddingStart="?android:attr/listPreferredItemPaddingStart"
    android:paddingEnd="?android:attr/listPreferredItemPaddingEnd"
    android:minHeight="?android:attr/listPreferredItemHeightSmall" />
```

## 定制界面

>**需求：让ListView的每个item左边放置一张图片，右边显示该图片的信息**

- 准备图片，创建drawable-xxhdpi目录，并将图片放入
- 定义实体类Fruit作为ListView适配器的适配类型

```kotlin
//类+构造函数+setter和getter方法,全部融为这一行代码
class Fruit(val name: String, val imageId: Int)
```

- 为ListView的子项准备自定义布局fruit_item

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="60dp"
    tools:ignore="UseCompoundDrawables">

    <ImageView
        android:id="@+id/fruitImage"
        android:layout_width="40dp"
        android:layout_height="40dp"
        android:layout_gravity="center_vertical"
        android:layout_marginLeft="10dp"
        android:contentDescription="@string/imgDes_1"
        android:layout_marginStart="10dp">
    </ImageView>

    <TextView
        android:id="@+id/fruitName"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center_vertical"
        android:layout_marginLeft="200dp"
        android:layout_marginStart="200dp">
    </TextView>
</LinearLayout>
```

- 适配器：继承ArrayAdapter适配器并重新getView方法

```kotlin
class FruitAdapter(activity : Activity, private val resourceId : Int, data : List<Fruit>) :
ArrayAdapter<Fruit>(activity,resourceId,data) {
    //ViewHolder类对ImageView和TextView控件进行缓存
    inner class ViewHolder(val fruitImage : ImageView,val fruitName : TextView)

    //convertView参数会对布局进行缓存
    override fun getView(position: Int, convertView: View?, parent: ViewGroup): View {
        val view : View
        val viewHolder : ViewHolder
        if (convertView == null){
            //没有缓存说明item是第一次出现,那就加载该item的布局
            view = LayoutInflater.from(context).inflate(resourceId, parent, false)
            //加载布局的同时找到布局中的控件封装到ViewHolder中进行缓存
            //这一步是为了减少查找布局里的控件的id的次数,就是保持了控件的id
            val fruitImage :ImageView = view.findViewById(R.id.fruitImage)
            val fruitName :TextView = view.findViewById(R.id.fruitName)
            viewHolder = ViewHolder(fruitImage,fruitName)
            //把ViewHolder对象存储在view中
            view.tag = viewHolder
        }else{
            //一个ListView子项就是一个view,向上转型,多态
            view = convertView
            viewHolder = view.tag as ViewHolder
        }
        //根据item的位置从传入的data数组中获取Fruit对象
        val fruit = getItem(position)
        if (fruit != null){ //data数组有多少个数据,就会有多少个item
            
            //将viewHolder中保存的控件取出并设置对应控件的资源
            viewHolder.fruitImage.setImageResource(fruit.imageId)
            viewHolder.fruitName.text = fruit.name;
        }
        //返回view,让用户可见
        return view
    }
}
```

- getView方法

>- 有多少个item就会调用多少次getView方法，在item滑动到可见位置使getView方法就调用
>- position从上往下，从0开始作为每个item的位置
>- 一开始能看到的item数量有多少，convertView就会缓存多少，之后的布局都是之前缓存的

- MainActivity

```kotlin
class MainActivity : AppCompatActivity() {
    private val fruitList = ArrayList<Fruit>()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        //1.初始化数据：添加Fruit对象到动态数组中
        initFruits()
        //2.创建适配器对象
        //参数1:当前上下文对象this   参数2:ListView子项布局 参数3:数据
        val adapter = FruitAdapter(this,R.layout.fruit_item,fruitList)
        //3.设置适配器
        list_view.adapter = adapter
        //4.为ListView设置监听器(可选)
        list_view.setOnItemClickListener{
            //parent, view, position -> 只有position参数被使用，其他参数可用下划线代替
            _, _, position, _ ->
            val fruit = fruitList[position]
            Toast.makeText(this,"你点击了"+fruit.name,Toast.LENGTH_SHORT).show()
        }
    }
    private fun initFruits(){
        //repeat(2)是一个函数，参数填入2，就会重复执行2次该方法
        repeat(2){
            fruitList.add(Fruit("apple",R.drawable.apple_pic))
            fruitList.add(Fruit("banana",R.drawable.banana_pic))
            fruitList.add(Fruit("cherry",R.drawable.cherry_pic))
            fruitList.add(Fruit("grape",R.drawable.grape_pic))
            fruitList.add(Fruit("mango",R.drawable.mango_pic))
            fruitList.add(Fruit("orange",R.drawable.orange_pic))
            fruitList.add(Fruit("pear",R.drawable.pear_pic))
            fruitList.add(Fruit("strawberry",R.drawable.strawberry_pic))
            fruitList.add(Fruit("pineapple",R.drawable.pineapple_pic))
            fruitList.add(Fruit("watermelon",R.drawable.watermelon_pic))
        }
    }
}
```

# RecyclerView

