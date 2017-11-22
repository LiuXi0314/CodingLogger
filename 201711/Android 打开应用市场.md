# Android 打开应用市场

标签（空格分隔）： Android 应用市场

---

从app跳转至应用商城里的app详情页

```java
      try {
                Intent i = new Intent(Intent.ACTION_VIEW);
                i.setData(Uri.parse("market://details?id="+getContext().getPackageName()));
                startActivity(i);
            } catch (Exception e) {
                ToastUtils.show("您的手机上没有安装Android应用市场");
                e.printStackTrace();
            }
```




