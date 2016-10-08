原文地址：[http://android.xsoftlab.net/training/multiple-threads/communicate-ui.html](http://android.xsoftlab.net/training/multiple-threads/communicate-ui.html)

上节课我们学习了如何启动一项由ThreadPoolExecutor所管理的线程任务。最后这节课我们将学习如何从任务中发送结果数据给UI线程。这项手段可以使任务在执行完毕后将结果显示到UI中去。

每个APP拥有独立的UI线程。只有在UI线程中创建的对象才可以访问该线程中的其它对象。正因为运行任务的线程不是UI线程，所以它们不可以直接访问UI对象。为了将数据从后台线程转移到UI线程，需要使用运行在UI线程中的Handler对象。

##在UI线程中定义Handler
Handler是Android系统框架管理线程的一部分。Handler对象专门用于接收消息处理消息。 一般来说，可以为新线程创建一个Handler，也可以为一个已经连接好的线程创建Handler。当你将Handler连接到UI线程时，处理消息的代码都会运行在UI线程中。

在构建线程池的类的构造方法中实例化一个Handler对象，并将该对象的引用存储于全局变量中。通过Handler(Looper)重载构造方法所实例化Handler可以与UI线程产生关联。这个构造方法所使用的Looper参数是Android系统的线程管理框架的另一部分。当以指定的Looper实例初始化一个Handler对象时，Handler对象会运行在Looper对象所在的线程中。
```java
private PhotoManager() {
...
    // Defines a Handler object that's attached to the UI thread
    mHandler = new Handler(Looper.getMainLooper()) {
    ...
```

在Handler内，重写handleMessage()方法。Android系统会在Handler所管理的线程中接收到一条新的消息时回调该方法：
```java
        /*
         * handleMessage() defines the operations to perform when
         * the Handler receives a new Message to process.
         */
        @Override
        public void handleMessage(Message inputMessage) {
            // Gets the image task from the incoming Message object.
            PhotoTask photoTask = (PhotoTask) inputMessage.obj;
            ...
        }
    ...
    }
}
```
##将数据从非UI线程转移到UI线程
为了将数据从后台进程转移到UI进程，首先将数据的引用以及UI对象存储于任务对象中。接下来，将该任务对象及状态码传给由Handler对象所初始化的对象。在这个对象内，发送一条包含状态码的任务对象给Handler。因为Handler是运行在UI线程的，所以它可以将数据交给UI对象。

###存储数据于任务对象中
举个例子，这里有一个Runnable对象，运行于后台线程，它用于解码一个Bitmap对象，并将其存储于它所属的对象PhotoTask中。该Runnable还会存储一个状态码：DECODE_STATE_COMPLETED。
```java
// A class that decodes photo files into Bitmaps
class PhotoDecodeRunnable implements Runnable {
    ...
    PhotoDecodeRunnable(PhotoTask downloadTask) {
        mPhotoTask = downloadTask;
    }
    ...
    // Gets the downloaded byte array
    byte[] imageBuffer = mPhotoTask.getByteBuffer();
    ...
    // Runs the code for this task
    public void run() {
        ...
        // Tries to decode the image buffer
        returnBitmap = BitmapFactory.decodeByteArray(
                imageBuffer,
                0,
                imageBuffer.length,
                bitmapOptions
        );
        ...
        // Sets the ImageView Bitmap
        mPhotoTask.setImage(returnBitmap);
        // Reports a status of "completed"
        mPhotoTask.handleDecodeState(DECODE_STATE_COMPLETED);
        ...
    }
    ...
}
...
```

PhotoTask还包含了一个用于展示Bitmap的ImageView的句柄。尽管引用Bitmap以及ImageView的是同一个对象，但是还是不能将Bitmap赋值给ImageView，因为当前并没有处在UI线程中。

##发送状态到对象层级
PhotoTask在层级内处于第二高度。它维护了图像的解码数据以及一个View对象。它会接收PhotoDecodeRunnable中的状态码，并将其传给维护线程池以及Handler的那个对象。
```java
public class PhotoTask {
    ...
    // Gets a handle to the object that creates the thread pools
    sPhotoManager = PhotoManager.getInstance();
    ...
    public void handleDecodeState(int state) {
        int outState;
        // Converts the decode state to the overall state.
        switch(state) {
            case PhotoDecodeRunnable.DECODE_STATE_COMPLETED:
                outState = PhotoManager.TASK_COMPLETE;
                break;
            ...
        }
        ...
        // Calls the generalized state method
        handleState(outState);
    }
    ...
    // Passes the state to PhotoManager
    void handleState(int state) {
        /*
         * Passes a handle to this task and the
         * current state to the class that created
         * the thread pools
         */
        sPhotoManager.handleState(this, state);
    }
    ...
}
```

###将数据展示到UI
PhotoManager收到从PhotoTask中发来的状态码，然后处理这个PhotoTask对象。因为状态是TASK_COMPLETE，所以先创建一个Message，然后通过这个Message对象将状态以及人物对象发送给Handler:
```java
public class PhotoManager {
    ...
    // Handle status messages from tasks
    public void handleState(PhotoTask photoTask, int state) {
        switch (state) {
            ...
            // The task finished downloading and decoding the image
            case TASK_COMPLETE:
                /*
                 * Creates a message for the Handler
                 * with the state and the task object
                 */
                Message completeMessage =
                        mHandler.obtainMessage(state, photoTask);
                completeMessage.sendToTarget();
                break;
            ...
        }
        ...
    }
```

最后，在Handler.handleMessage()中检查每条消息的状态。如果消息的状态为TASK_COMPLETE，那么表明该任务已经终结，那么Message中所包含的PhotoTask对象包含了一个Bitmap以及一个ImageView。因为Handler.handleMessage()是运行在UI线程的，所以它可以安全的将Bitmap赋给ImageView:
```java
    private PhotoManager() {
        ...
            mHandler = new Handler(Looper.getMainLooper()) {
                @Override
                public void handleMessage(Message inputMessage) {
                    // Gets the task from the incoming Message object.
                    PhotoTask photoTask = (PhotoTask) inputMessage.obj;
                    // Gets the ImageView for this task
                    PhotoView localView = photoTask.getPhotoView();
                    ...
                    switch (inputMessage.what) {
                        ...
                        // The decoding is done
                        case TASK_COMPLETE:
                            /*
                             * Moves the Bitmap from the task
                             * to the View
                             */
                            localView.setImageBitmap(photoTask.getImage());
                            break;
                        ...
                        default:
                            /*
                             * Pass along other messages from the UI
                             */
                            super.handleMessage(inputMessage);
                    }
                    ...
                }
                ...
            }
            ...
    }
...
}
```

