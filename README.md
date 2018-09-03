[TOC]

# 1 ToastShowWrapper

Can show toast without notification permission.
Inspired by [EToast2](https://github.com/Blincheng/EToast2)

## 1.1 Source Code

```java
    private static class ToastShowWrapper{
        private WindowManager manger;
        private View contentView;
        private WindowManager.LayoutParams params;
        private Handler handler;
        static final long SHORT_DURATION_TIMEOUT = 4000;
        static final long LONG_DURATION_TIMEOUT = 7000;
        private static int WINDOW_TYPE = -1;
        private static int APP_WINDOW_TYPE = new WindowManager.LayoutParams().type;

        WrapperToast(){
            if(WINDOW_TYPE == -1) {
                if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
                    WINDOW_TYPE = WindowManager.LayoutParams.TYPE_APPLICATION_OVERLAY;
                } else {
                    WINDOW_TYPE = WindowManager.LayoutParams.TYPE_TOAST;
                }
            }
        }

        public void show(Context context, Toast toast){
            if(isNotificationEnabled(context)){
                toast.show();
            }else {
                showInWindow(context, toast);
            }
        }

        private void showInWindow(Context context, Toast toast){
            hideInWindow();

            manger = (WindowManager) context.getSystemService(Context.WINDOW_SERVICE);


            if(handler == null){
                handler = new Handler(Looper.getMainLooper());
            }

            contentView = toast.getView();
            params = new WindowManager.LayoutParams();
            params.height = WindowManager.LayoutParams.WRAP_CONTENT;
            params.width = WindowManager.LayoutParams.WRAP_CONTENT;
            params.format = PixelFormat.TRANSLUCENT;
            params.windowAnimations = -1;
            params.setTitle("WrapperToast");
            params.flags = WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON
                    | WindowManager.LayoutParams.FLAG_NOT_FOCUSABLE
                    | WindowManager.LayoutParams.FLAG_NOT_TOUCHABLE;
            params.gravity = toast.getGravity();
            params.x = toast.getXOffset();
            params.y = toast.getYOffset();
            params.verticalMargin = toast.getVerticalMargin();
            params.horizontalMargin = toast.getHorizontalMargin();
            params.type = WINDOW_TYPE;
            try {
                manger.addView(contentView, params);
            }catch (Exception e){//
                if(params.type == APP_WINDOW_TYPE){
                    e.printStackTrace();
                }else {//OPPO R11s Android 7.1.1
                    WINDOW_TYPE = APP_WINDOW_TYPE;
                    params.type = APP_WINDOW_TYPE;
                    try {
                        manger.addView(contentView, params);
                    } catch (Exception e1) {
                        e1.printStackTrace();
                    }
                }
            }

            handler.postDelayed(new Runnable() {
                @Override
                public void run() {
                    hideInWindow();
                }
            }, toast.getDuration()==Toast.LENGTH_SHORT? SHORT_DURATION_TIMEOUT:LONG_DURATION_TIMEOUT);
        }

        private void hideInWindow(){
            if(manger != null && contentView != null && contentView.getParent() != null){
                try {
                    manger.removeView(contentView);
                    contentView = null;
                } catch (IllegalArgumentException e) {
                    e.printStackTrace();
                }
            }
        }

        //refer to https://github.com/Blincheng/EToast2
        private boolean isNotificationEnabled(Context context){
            if(android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.KITKAT){
                AppOpsManager mAppOps = (AppOpsManager) context.getSystemService(Context.APP_OPS_SERVICE);
                ApplicationInfo appInfo = context.getApplicationInfo();

                String pkg = context.getApplicationContext().getPackageName();

                int uid = appInfo.uid;

                Class appOpsClass = null; /* Context.APP_OPS_MANAGER */

                try {
                    appOpsClass = Class.forName(AppOpsManager.class.getName());
                    Method checkOpNoThrowMethod = appOpsClass.getMethod("checkOpNoThrow", Integer.TYPE, Integer.TYPE, String.class);
                    Field opPostNotificationValue = appOpsClass.getDeclaredField("OP_POST_NOTIFICATION");
                    int value = (int)opPostNotificationValue.get(Integer.class);
                    return ((int)checkOpNoThrowMethod.invoke(mAppOps,value, uid, pkg) == AppOpsManager.MODE_ALLOWED);
                } catch (Exception e) {
                    e.printStackTrace();
                    return false;
                }
            }else{
                return false;
            }
        }
    }
```



## 1.2 Usage

Create a single instance of ToastShowWrapper, show android Toast use ToastShowWrapper.show.

```java
//Global instance
public final static ToastShowWrapper TOAST_WRAPPER = new ToastShowWrapper();

...
  
//show toast in somewhere
TOAST_WRAPPER.show(context/*Android context*/, toast/*android.widget.Toast*/);

```
