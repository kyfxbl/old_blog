title: android process and thread
date: 2013-09-24 10:33
categories: android 
---
android中近程和线程的有关知识点，11年的时候写的，不知道现在是否已经过时了。通过在manifest中配置，是可以改变android系统的默认行为的，不过本文只总结默认配置下的情况
<!--more-->

1. 一个application对应一个process 

2. 所有的component（包括activity，service）都跑在同一个thread中，这个thread是main线程，也叫UI thread。也就是如果应用不做额外的编码，所有的组件都跑在同一个线程中

3. android还是比较好心的，it tries to maintain an application process for as long as possible 

4. 但在资源不够的情况下，android系统就开始kill process了。优先级包括，foreground process、visible process、service process、background process、empty process。从后往前依次kill 

5. 进程之间有依赖关系的话，process that is serving another process can never be ranked lower than the process it is serving 

6. 由于前面2提到的，默认所有组件都跑在UI thread里，页面UI响应会很不好，所以要编码解决这个问题。需要遵循2条规则： 

A. 不能阻塞UI thread 
B. 不要在UI thread之外访问UI组件

```
public void onClick(View v) {
    new Thread(new Runnable() {
        public void run() {
            Bitmap b = loadImageFromNetwork("http://example.com/image.png");
            mImageView.setImageBitmap(b);
        }
    }).start();
}
```

上面的代码就违反了规则2，在worker thread中访问了UI组件mImageView
```
public void onClick(View v) {
    new Thread(new Runnable() {
        public void run() {
            final Bitmap bitmap = loadImageFromNetwork("http://example.com/image.png");
            mImageView.post(new Runnable() {
                public void run() {
                    mImageView.setImageBitmap(bitmap);
                }
            });
        }
    }).start();
}
```

上面的代码就比较好了，但是可以用AsyncTask来进一步优化
```
public void onClick(View v) {
    new DownloadImageTask().execute("http://example.com/image.png");
}

private class DownloadImageTask extends AsyncTask<String, Void, Bitmap> {

    protected Bitmap doInBackground(String... urls) {
        return loadImageFromNetwork(urls[0]);
    }

    protected void onPostExecute(Bitmap result) {
        mImageView.setImageBitmap(result);
    }
}
```