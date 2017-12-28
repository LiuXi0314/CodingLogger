# Android 基于AppCompatAutoCompleteTextView实现的自动补全邮箱地址控件

标签：Android  Email  AutoCompleteTextView

---
> 近日我司工作上有需求：在用户登录注册时 **自动补全用户输入的邮箱账号**（本产品服务于外国友人，所以仅支持邮箱注册），以便于用户操作，给用户一个良好体验。拿到手后感觉并没有什么难度，便火急火燎的去实现了，哪想到一步一坑，克服了一些问题后才有了此篇文章中的劳动成果。因为在实现过程中搜索到的资源利用率不高，便于此记录一下，方便后人乘凉，
谨以此篇文章的记录下自己挖坑填坑之旅，且以后在实现需求时要时时告诫自己：**源码拜读一遍，可省三日之功**。

> 参考链接：http://my.oschina.net/fengheju/blog/176656?fromerr=vA0tbaoq

> 文章原址： https://github.com/LiuXi0314/CodingLogger/blob/master/201712/Android%20%E5%9F%BA%E4%BA%8EAppCompatAutoCompleteTextView%E5%AE%9E%E7%8E%B0%E7%9A%84%E8%87%AA%E5%8A%A8%E8%A1%A5%E5%85%A8%E9%82%AE%E7%AE%B1%E5%9C%B0%E5%9D%80%E6%8E%A7%E4%BB%B6.md 欢迎关注

本篇文章主要讲述邮箱地址自动补全的功能实现，对于AutoCompleteTextView的基本功能请自行百度或者阅读源码。
### 实现思路
1.利用AutoCompleteTextView + 自定义ArrayAdapter实现pop UI;
2.重写AutoCompleteTextView 的performFiltering方法，修改此方法传入的原始文本：当用户未输入 **@** 时，原始文本置换为 **@** ；在输入 **@** 后，原始文本置换为 原始文本的 **@及其之后的部分** ;当用户输入错误字符时关闭pop。
```java 
        @Override
        protected void performFiltering(CharSequence text, int keyCode) {
            String t = text.toString();
            int index = t.indexOf("@");
            if (index == -1) {
                if (t.matches("^[a-zA-Z0-9_]+$")) {
                    super.performFiltering("@", keyCode);
                } else
                    this.dismissDropDown();//关闭pop
            } else {
                super.performFiltering(t.substring(index), keyCode);
            }
        }
```

3.由于补全邮箱时我个人想实现的的功能是补全所有含有过滤文本的匹配内容，而不是所有以过滤文本开头的匹配内容，所以只能重写ArrayAdapter 的filter来实现此功能

```java
 private class ArrayFilter extends Filter {
            @Override
            protected FilterResults performFiltering(CharSequence prefix) {
                final FilterResults results = new FilterResults();
                if (prefix == null || prefix.length() == 0) {
                    final List<String> list;
                    synchronized (this) {
                        list = Arrays.asList(address.clone());
                    }
                    results.values = list;
                    results.count = list.size();
                } else {
                    String prefixString = prefix.toString().toLowerCase();
                    prefixString = prefixString.substring(1);
                    final ArrayList<String> values;
                    synchronized (this) {
                        values = new ArrayList<>(ConvertUtils.toList(address));
                    }
                    final int count = values.size();
                    final ArrayList<String> newValues = new ArrayList<>();

                    for (int i = 0; i < count; i++) {
                        final String value = values.get(i);
                        final String valueText = value.toString().toLowerCase();
                        // First match against the whole, non-splitted value
                        if (valueText.contains(prefixString)) {
                            newValues.add(value);
                        } else {
                            final String[] words = valueText.split(" ");
                            for (String word : words) {
                                if (word.contains(prefixString)) {
                                    newValues.add(value);
                                    break;
                                }
                            }
                        }
                    }

                    results.values = newValues;
                    results.count = newValues.size();
                }

                return results;
            }

            @Override
            protected void publishResults(CharSequence constraint, FilterResults results) {
                //noinspection unchecked
                clear();
                if (results.count > 0) {
                    for (String str : (List<String>) results.values){
                        add(str);
                    }
                    notifyDataSetChanged();
                } else {
                    notifyDataSetInvalidated();
                }
            }
        }
```
4.AutoCompleteTextView 在点击pop item后会将我们传入的原始值赋值给TextView，但在实现自动补全邮箱域名时，这个结果并不是我们想看到的。所以我们需要去重写AutoCompleteTextView的replaceText()方法，将我们自动补全后的邮箱地址赋值TextView.

```java
   @Override
    protected void replaceText(CharSequence text) {
        String t = this.getText().toString();
        //当我们在下拉框中选择一项时，android会默认使用AutoCompleteTextView中Adapter里的文本来填充文本域
        //因为这里Adapter中只是存了常用email的后缀
        //因此要重新replace逻辑，将用户输入的部分与后缀合并
        int index = t.indexOf("@");
        if (index != -1)
            t = t.substring(0, index);
        super.replaceText(t + text);

    }
```
5.在文本框获取焦点时唤出pop。
```java
   @Override
    protected void onFocusChanged(boolean hasFocus, int direction, Rect previouslyFocusedRect) {
        super.onFocusChanged(hasFocus, direction, previouslyFocusedRect);
        if (hasFocus) {
            String text = EmailAutoCompleteTextView.this.getText().toString();
            if (!"".equals(text))
                performFiltering(text, 0);
        }
    }

```

### 遇到的问题：

在重写Filter类时需要在publishResults(CharSequence constraint, FilterResults results)方法中将过滤后的结果results赋值到Adapter的origin Value中，修改完此处代码时调试运行，一直遇到一个异常：

```java
E/UncaughtException: java.lang.UnsupportedOperationException  

at java.util.AbstractList.remove(AbstractList.java:161)                                                 
at java.util.AbstractList$Itr.remove(AbstractList.java:374)
at java.util.AbstractList.removeRange(AbstractList.java:571)                                               at java.util.AbstractList.clear(AbstractList.java:234)
at android.widget.ArrayAdapter.clear(ArrayAdapter.java:320)
at com.xxxxx.EmailAutoCompleteTextView$EmailAdapter$ArrayFilter.publishResults(EmailAutoCompleteTextView.java:200)
at android.widget.Filter$ResultsHandler.handleMessage(Filter.java:282)
at android.os.Handler.dispatchMessage(Handler.java:105)
at android.os.Looper.loop(Looper.java:164)
at android.app.ActivityThread.main(ActivityThread.java:6809)
at java.lang.reflect.Method.invoke(Native Method)
at com.android.internal.os.Zygote$MethodAndArgsCaller.run(Zygote.java:240)
at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:767)
```

查询一些帖子才发现造成原因是：在重写filter方法中对adapter使用了 add() remove() 等操作，传入的List 为**Arrays.asList(address)**,但是在使用Arrays.asLisvt()后调用add，remove这些method时会出现java.lang.UnsupportedOperationException异常。
这是由于：
Arrays.asLisvt() 返回java.util.Arrays.ArrayList， 而不是ArrayList。Arrays.ArrayList和ArrayList都是继承AbstractList，remove，add等 method在AbstractList中是默认throw UnsupportedOperationException而且不作任何操作。ArrayList override这些method来对list进行操作，但是Arrays$ArrayList没有override remove(int)，add(int)等，所以throw UnsupportedOperationException。

解决方法是使用Iterator，或者转换为ArrayList

```java
List list = Arrays.asList(a[]);
List arrayList = new ArrayList(list);

or

public static <T> List<T> toList(T[] array) {
    List<T> tmpList = new ArrayList<T>();
    if (array == null) return tmpList;
    for (T item : array) {
        tmpList.add(item);
    }
    return tmpList;
}
```


### 完整源码如下：
```java
package com.xxxxxxx.view;

import android.content.Context;
import android.graphics.Rect;
import android.support.annotation.NonNull;
import android.support.annotation.Nullable;
import android.support.v7.widget.AppCompatAutoCompleteTextView;
import android.util.AttributeSet;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.ArrayAdapter;
import android.widget.Filter;
import android.widget.TextView;

import com.xxxx.xxx.R;
import com.xxxxxx.library.utils.ConvertUtils;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

/**
 * 根据用户输入的内容自动补全邮箱地址
 * Created on 17-12-26 下午4:20 by LiuXi0314
 */

public class EmailAutoCompleteTextView extends AppCompatAutoCompleteTextView {

    private String[] address = new String[]{
            "@gmail.com",
            "@yahoo.com",
            "@hotmail.com",
            "@outlook.com",
            "@aol.com",
            "@excite.com",
            "@icloud.com",
            "@ideakings.com",
            "@juno.com",
            "@langevinri.com",
            "@matpost.com",
            "@netzero.com",
            "@tradesult.com",
            "@tds.net",
            "@verizon.net",
            "@windstream.net",
            "@att.net",
            "@bellsouth.net",
            "@charter.net",
            "@comcast.net",
            "@gci.net",};

    public EmailAutoCompleteTextView(Context context) {
        super(context);
        initViews();
    }

    public EmailAutoCompleteTextView(Context context, AttributeSet attrs) {
        super(context, attrs);
        initViews();
    }

    public EmailAutoCompleteTextView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        initViews();
    }

    private void initViews() {
        setAdapter(new EmailAdapter(ConvertUtils.toList(address)));
        setThreshold(1);	//使得在输入1个字符之后便弹出pop
    }


    @Override
    protected void onFocusChanged(boolean hasFocus, int direction, Rect previouslyFocusedRect) {
        super.onFocusChanged(hasFocus, direction, previouslyFocusedRect);
        if (hasFocus) {
            String text = EmailAutoCompleteTextView.this.getText().toString();
            if (!"".equals(text))
                performFiltering(text, 0);
        }
    }

    @Override
    protected void replaceText(CharSequence text) {
        String t = this.getText().toString();
        //当我们在下拉框中选择一项时，android会默认使用AutoCompleteTextView中Adapter里的文本来填充文本域
        //因为这里Adapter中只是存了常用email的后缀
        //因此要重新replace逻辑，将用户输入的部分与后缀合并
        int index = t.indexOf("@");
        if (index != -1)
            t = t.substring(0, index);
        super.replaceText(t + text);

    }

    @Override
    protected void performFiltering(CharSequence text, int keyCode) {
        //该方法会在用户输入文本之后调用，将已输入的文本与adapter中的数据对比，若它匹配
        //adapter中数据的前半部分，那么adapter中的这条数据将会在下拉框中出现
        String t = text.toString();
        //因为用户输入邮箱时，都是以字母，数字开始，而我们的adapter中只会提供以类似于"@163.com"
        //的邮箱后缀，因此在调用super.performFiltering时，传入的一定是以"@"开头的字符串
        int index = t.indexOf("@");
        if (index == -1) {
            if (t.matches("^[a-zA-Z0-9_]+$")) {
                super.performFiltering("@", keyCode);
            } else
                this.dismissDropDown();//当用户中途输入非法字符时，关闭下拉提示框
        } else {
            super.performFiltering(t.substring(index), keyCode);
        }
    }


    class EmailAdapter extends ArrayAdapter<String>{

        public EmailAdapter(@NonNull List<String> objects) {
            super(EmailAutoCompleteTextView.this.getContext(),R.layout.item_email_auto_complete , R.id.textView, objects);
        }

        @NonNull
        @Override
        public View getView(int position, @Nullable View convertView, @NonNull ViewGroup parent) {
            View v = convertView;
            if (v == null)
                v = LayoutInflater.from(getContext()).inflate(
                        R.layout.item_email_auto_complete, null);
            String originStr = EmailAutoCompleteTextView.this.getText().toString();
            int index = originStr.indexOf("@");
            if (index != -1)
                originStr = originStr.substring(0, index);
            final TextView textView = (TextView) v.findViewById(R.id.textView);
            textView.setText(originStr + getItem(position));
            return textView;
        }

        @NonNull
        @Override
        public Filter getFilter() {
            return new ArrayFilter();
        }

        /**
         * 修改ArrayAdapter源码 中的ArrayFilter，以改变其过滤规则
         * 默认过滤规则为 startWith == true
         * 修改后的规则： contains == true
         */
        private class ArrayFilter extends Filter {
            @Override
            protected FilterResults performFiltering(CharSequence prefix) {
                final FilterResults results = new FilterResults();
                if (prefix == null || prefix.length() == 0) {
                    final List<String> list;
                    synchronized (this) {
                        list = Arrays.asList(address.clone());
                    }
                    results.values = list;
                    results.count = list.size();
                } else {
                    String prefixString = prefix.toString().toLowerCase();
                    prefixString = prefixString.substring(1);
                    final ArrayList<String> values;
                    synchronized (this) {
                        values = new ArrayList<>(ConvertUtils.toList(address));
                    }
                    final int count = values.size();
                    final ArrayList<String> newValues = new ArrayList<>();

                    for (int i = 0; i < count; i++) {
                        final String value = values.get(i);
                        final String valueText = value.toString().toLowerCase();
                        // First match against the whole, non-splitted value
                        if (valueText.contains(prefixString)) {
                            newValues.add(value);
                        } else {
                            final String[] words = valueText.split(" ");
                            for (String word : words) {
                                if (word.contains(prefixString)) {
                                    newValues.add(value);
                                    break;
                                }
                            }
                        }
                    }

                    results.values = newValues;
                    results.count = newValues.size();
                }

                return results;
            }

            @Override
            protected void publishResults(CharSequence constraint, FilterResults results) {
                //noinspection unchecked
                clear();
                if (results.count > 0) {
                    for (String str : (List<String>) results.values){
                        add(str);
                    }
                    notifyDataSetChanged();
                } else {
                    notifyDataSetInvalidated();
                }
            }
        }

    }
}

```




