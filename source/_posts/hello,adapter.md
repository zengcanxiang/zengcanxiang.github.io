---
date:
  - 2016-9-01
tags:
    - Android
    - 笔记
    - hello
title: hello，adapter
---
Android适配器使用经验总结.
<!--more-->
## BaseAdapter
　如果问我们Android开发人员使用的控件最多的是那一个？可能答案有很多种,几分天下,但是我想，ListView应该是其中的霸主吧.  
　从一开始敲出的"Hello,word!"到绚丽的APP,中间,我们经历了多少的ListView?写了多少的Adapter?  
　想想，那些成长的日子,越写越简单的Adapter,越来越流畅的ListView,是多好的见证.  
　庆幸,我比较早的碰上了[hongyang][1]的base-adapter-helper.
* [视频][2]
* [Github项目][3]
* [源码分析文章][4]

　当我还在一遍又一遍重写BaseAdapter的getCount、getItem、getItemId等方法，一遍又一遍的在getView方法里面inflate Layout,一遍又一遍的将ViewHolder setTag,getTag.在一开始走上自学不久的时候，就看到了很精彩的内容.  
　新手自然推荐看视频,洋神的声音还带点磁性~当初在一通乱点的视频中,我就是因为这个声音而停下了脚步.  
　因为也是算是学习笔记，就直接从自身出发了. 关于baseAdapter不管是洋神的视频还是文章都不知道比我的水平高出多少了.
　我维护了一个项目在Github上面，是关于Adapter.在有别人的项目的时候，我为什么要自己写一个呢。  
 * 第一点，当初这个项目是为了照着视频上打的,学习的而产生的.一开始,洋神并不会把代码放出来,自然只能自己手敲,不停的踏入无数的坑中.但是，这样才不会让更多的人变成伸手党.感谢洋神~  
 * 第二点，洋神的代码与我的代码规范或者思路有点不同，为了成长，我要实现自己的思路，并且改进他.才不会写出想当然的代码，也能更清晰的了解洋神代码的思路和设计.
 * 第三点，当初在RecyclerView出来的时候，洋神并没有出视频或者更新代码，就自己尝试写了。并且，有区别于别人的Adapter项目的是，我自己琢磨写了ExpandableListView、ViewPager的BaseAdapter.  

　然后介绍一下我的思路.
  * 我将代码通过包名来分层,一般直接继承的叫---BaseAdapter,继承自定义的BaseAdapter,为他添加简便方法的叫HelperAdapter.导入的时候，通过import语句来区分.  
  而且将HelperAdapter和BaseAdapter区分开来，假如需要自己来实现一些特殊的代码，也不会需要导入源码来修改.我个人不喜欢出现很长名字的方法和类名，这样也避免了出现很多乱七八糟的类名.
  * 第二个思路就是多子布局的设计.这个完全就是自己设计的,一开始洋神的视频和代码都没有提到这个,即使是后来更新了代码，也是完全不一样的想法和思路.  
  　　产生了这个想法,也是因为当时公司的项目有一个界面,itemType比较复杂,实现效果时写了很多if-else-if 的丑陋代码.虽然后来出现了很多多子布局BaseAdapter.但是我比较了代码和提交时间，采取int...和复写父类方法来动态配置layoutId,我应该是走的比较早.  
  　　因为这个想法，是通过学习V4-Support的SwipeRefreshLayout设置圆形圈颜色而产生的.这对我来说,使我的编程能力得到了成长(并不是编码能力).

　先给大家看一下效果.毕竟无图那个是吧。  
　![示例](http://o6y6mbc2m.bkt.clouddn.com/Animation.gif)


## 使用方法
下面通过一些代码来简单介绍一下如何使用.因为代码使用规范差不多，重复的就不一一贴出来了.
### 普通absListView
```java
import com.zengcanxiang.baseAdapter.absListView.HelperAdapter;
import com.zengcanxiang.baseAdapter.absListView.HelperViewHolder;

private class ExampleListAdapter extends HelperAdapter<String> {

      public ExampleListAdapter(List<String> mList, Context context, int... layoutIds) {
          super(mList, context, layoutIds);
          //这样是处理不需要动态配置layoutId的情况，多个也可如此配置
          //super(mList, context, R.layot.xxxx，R.layout.xxxx);

      }

      @Override
      public void HelpConvert(HelperViewHolder viewHolder, int position, String s) {

      }
  }
```
#### 多子布局的recyclerView
```java
import com.zengcanxiang.baseAdapter.recyclerView.HelperAdapter;
import com.zengcanxiang.baseAdapter.recyclerView.HelperViewHolder;

private class ExampleRecyAdapter extends HelperAdapter<Msg> {
       public ExampleRecyAdapter(List<Msg> data, Context context) {
           super(data, context, R.layout.example_exrecycler_1,R.layout.example_recycler_2);
       }
       @Override
       protected void HelperBindData(HelperViewHolder viewHolder, int position, Msg item) {
           switch (item.getType()) {
               case 0:
                   break;
               case 1:
                   break;
           }
       }

       @Override
       public int checkLayoutIndex(Msg item, int position) {
           /**
            * 多子布局样式重写checkLayout()方法，返回对应的index
            * 本例子因为msg的Type对应的就是0和1,所以就直接返回msgType
            */
           return item.getType();
       }
   }
```
#### 普通expandableListView
```java
import com.zengcanxiang.baseAdapter.absListView.HelperViewHolder;
import com.zengcanxiang.baseAdapter.expandableListView.HelperAdapter;

/**
*标题数据与子数据在一个实体
*/
private class ExampleAdapter extends HelperAdapter<Msg> {

        public ExampleAdapter(List<List<Msg>> data, Context context, int[] groupLayoutIds, int... childLayoutIds) {
            super(data, context, groupLayoutIds, childLayoutIds);
        }

        @Override
        public void HelpConvertGroup(HelperViewHolder viewHolder, int groupPosition, List<Msg> childs) {
            viewHolder.setText(R.id.expandable_group, childs.get(0).getGroupMsg());
        }

        @Override
        public void HelpConvertChild(HelperViewHolder viewHolder, int groupPosition, int childPosition, Msg msg) {
            viewHolder.setText(R.id.example_item_text_view, msg.getMsg());
        }
}

/**
*标题数据与子数据不在一个实体
*/
private class ExampleAdapter2 extends HelperAdapter2<GroupMsg, Msg> {

        public ExampleAdapter2(List<GroupMsg> groupData, List<List<Msg>> childData, Context context, int[] groupLayoutIds, int... childLayoutIds) {
            super(groupData, childData, context, groupLayoutIds, childLayoutIds);
        }

        @Override
        public void HelpConvertGroup(HelperViewHolder viewHolder, int groupPosition, GroupMsg item, List<Msg> childs) {
            viewHolder.setText(R.id.expandable_group, item.getMsg());
        }

        @Override
        public void HelpConvertChild(HelperViewHolder viewHolder, int groupPosition, int childPosition, Msg msg) {
            viewHolder.setText(R.id.example_item_text_view, msg.getMsg());
        }
    }
```
#### ViewPager:使用view和使用fragment

```java

import com.zengcanxiang.baseAdapter.viewpager.BaseAdapter;
import com.zengcanxiang.baseAdapter.viewpager.BaseFragmentAdapter;

public class ViewAdapter extends BaseAdapter {

      public ViewAdapter(Context context) {
          super(context, R.layout.example_viewpager_one,R.layout.example_viewpager_two,R.layout.example_viewpager_three);
      }

      @Override
      public void convert(View view, int position) {

      }
  }

  public class FragmentAdapter extends BaseFragmentAdapter {

      public FragmentAdapter(FragmentManager fm, @NonNull List<Fragment> fragments) {
          super(fm, fragments);
      }
  }

```

#### headFootViewAdapter(RecyclerView)
```java
import com.zengcanxiang.baseAdapter.recyclerView.HeadFootAdapter;

MyRecyerAdapter mAdapter = new MyRecyerAdapter(mList, this, R.layout.example_item);
HeadFootAdapter headFootAdapter = new HeadFootAdapter(mAdapter) {
            @Override
            public void disposeHeadView(HelperViewHolder viewHolder, int layoutId, int position) {
            }

            @Override
            public void disposeFootView(HelperViewHolder viewHolder, View footView, int position) {

            }
        };
//注意：推荐head或者foot不是listView或者之类的列表控件
//如果超出一屏的话，将他们分割开来，不要一次性写入到一个xml中
headFootAdapter.addHeadView(R.layout.list_head_home);
```
好了，就介绍到这里,程序员的世界还是用代码说话吧.[传送门][5]

---
[1]:https://github.com/hongyangAndroid/
[2]:http://www.imooc.com/learn/372
[3]:https://github.com/hongyangAndroid/base-adapter-helper
[4]:http://a.codekk.com/detail/Android/hongyangAndroid/BaseAdapterHelper%20%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90
[5]:https://github.com/zengcanxiang/BaseAdapter
