---
layout:     post
title:      " Choreographer"
subtitle:   "vsync处理"
date:       2021-07-03
author:     "zhouhaoh"
catalog: true
tags:
- Android
---

#### VSync机制

16毫秒一个信号，垂直同步机制，同步UI绘制和动画，使他们可以获得一个达到``60fps``的固定帧率。Choreographer负责这个机制，接收到后执行绘制流程。就算屏幕界面保持不变，当接收到``vsync``信号的时候，也会刷新屏幕。

#### Choreographer机制

消息处理器，根据vsync型号来计算帧的绘制，主要处理三种回调：

1. CALLBACK_INPUT
2. CALLBACK_ANIMATION
3. CALLBACK_TRAVERSAL

在``ViewRootImpl``中的``scheduleTraversals``方法中，会执行mTraversalRunnable的run方法。最终会执行view绘制流程，

```java
    void scheduleTraversals() {
        if (!mTraversalScheduled) {
            mTraversalScheduled = true;
            //发送同步屏障
            mTraversalBarrier = mHandler.getLooper().getQueue().postSyncBarrier();
            //执行了postCallBack，类型：CALLBACK_TRAVERSAL
            mChoreographer.postCallback(
                    Choreographer.CALLBACK_TRAVERSAL, mTraversalRunnable, null);
            if (!mUnbufferedInputDispatch) {
                scheduleConsumeBatchedInput();
            }
            notifyRendererOfFramePending();
            pokeDrawLockIfNeeded();
        }
    }
```

进入``postCallback``方法：

```java
  @TestApi
    public void postCallback(int callbackType, Runnable action, Object token) {
        postCallbackDelayed(callbackType, action, token, 0);
    }

```

```java
 @TestApi
    public void postCallbackDelayed(int callbackType,
            Runnable action, Object token, long delayMillis) {
        if (action == null) {
            throw new IllegalArgumentException("action must not be null");
        }
        if (callbackType < 0 || callbackType > CALLBACK_LAST) {
            throw new IllegalArgumentException("callbackType is invalid");
        }

        postCallbackDelayedInternal(callbackType, action, token, delayMillis);
    }

```

```java
private void postCallbackDelayedInternal(int callbackType,
            Object action, Object token, long delayMillis) {
        if (DEBUG_FRAMES) {
            Log.d(TAG, "PostCallback: type=" + callbackType
                    + ", action=" + action + ", token=" + token
                    + ", delayMillis=" + delayMillis);
        }

        synchronized (mLock) {
            final long now = SystemClock.uptimeMillis();
            final long dueTime = now + delayMillis;
            //****！！！！将action封装成CallbackRecord存储，后续会取出来执行。
            mCallbackQueues[callbackType].addCallbackLocked(dueTime, action, token);

            if (dueTime <= now) {
                scheduleFrameLocked(now);
            } else {
                Message msg = mHandler.obtainMessage(MSG_DO_SCHEDULE_CALLBACK, action);
                msg.arg1 = callbackType;
                msg.setAsynchronous(true);
                mHandler.sendMessageAtTime(msg, dueTime);
            }
        }
    }

```

根据以上代码：token=null，delayMillis=0，所以会进入if条件判断，进入``   scheduleFrameLocked(now);``方法：

```java
 private void scheduleFrameLocked(long now) {
        if (!mFrameScheduled) {
            mFrameScheduled = true;
            if (USE_VSYNC) {//默认为true
                if (DEBUG_FRAMES) {
                    Log.d(TAG, "Scheduling next frame on vsync.");
                }

                // If running on the Looper thread, then schedule the vsync immediately,
                // otherwise post a message to schedule the vsync from the UI thread
                // as soon as possible.
                if (isRunningOnLooperThreadLocked()) {
                    scheduleVsyncLocked();
                } else {
                    Message msg = mHandler.obtainMessage(MSG_DO_SCHEDULE_VSYNC);
                    msg.setAsynchronous(true);
                    mHandler.sendMessageAtFrontOfQueue(msg);
                }
            } else {
                final long nextFrameTime = Math.max(
                        mLastFrameTimeNanos / TimeUtils.NANOS_PER_MS + sFrameDelay, now);
                if (DEBUG_FRAMES) {
                    Log.d(TAG, "Scheduling next frame in " + (nextFrameTime - now) + " ms.");
                }
                Message msg = mHandler.obtainMessage(MSG_DO_FRAME);
                msg.setAsynchronous(true);
                mHandler.sendMessageAtTime(msg, nextFrameTime);
            }
        }
    }
```

最终会判断looper的线程，执行``scheduleVsyncLocked()``方法：

```java
 private void scheduleVsyncLocked() {
        mDisplayEventReceiver.scheduleVsync();
    }
```

通过DisplayEventReceiver调用native的方法：最终回调在onVsync方法：

```java
    // Called from native code.
    @SuppressWarnings("unused")
    private void dispatchVsync(long timestampNanos, int builtInDisplayId, int frame) {
        onVsync(timestampNanos, builtInDisplayId, frame);
    }

```

而在onVsync方法中，发送了一个Message，mDisplayEventReceiver实现了Runnable，会执行mDisplayEventReceiver的run方法：

```java
 Message msg = Message.obtain(mHandler, this);
msg.setAsynchronous(true);
```

```java
 @Override
        public void run() {
            mHavePendingVsync = false;
            doFrame(mTimestampNanos, mFrame);
        }
```

```java
  void doFrame(long frameTimeNanos, int frame) {
        final long startNanos;
        synchronized (mLock) {
            if (!mFrameScheduled) {
                return; // no work to do
            }

            if (DEBUG_JANK && mDebugPrintNextFrameTimeDelta) {
                mDebugPrintNextFrameTimeDelta = false;
                Log.d(TAG, "Frame time delta: "
                        + ((frameTimeNanos - mLastFrameTimeNanos) * 0.000001f) + " ms");
            }

            long intendedFrameTimeNanos = frameTimeNanos;
            startNanos = System.nanoTime();
            final long jitterNanos = startNanos - frameTimeNanos;
            if (jitterNanos >= mFrameIntervalNanos) {
                final long skippedFrames = jitterNanos / mFrameIntervalNanos;
                if (skippedFrames >= SKIPPED_FRAME_WARNING_LIMIT) {
                    Log.i(TAG, "Skipped " + skippedFrames + " frames!  "
                            + "The application may be doing too much work on its main thread.");
                }
                final long lastFrameOffset = jitterNanos % mFrameIntervalNanos;
                if (DEBUG_JANK) {
                    Log.d(TAG, "Missed vsync by " + (jitterNanos * 0.000001f) + " ms "
                            + "which is more than the frame interval of "
                            + (mFrameIntervalNanos * 0.000001f) + " ms!  "
                            + "Skipping " + skippedFrames + " frames and setting frame "
                            + "time to " + (lastFrameOffset * 0.000001f) + " ms in the past.");
                }
                frameTimeNanos = startNanos - lastFrameOffset;
            }

            if (frameTimeNanos < mLastFrameTimeNanos) {
                if (DEBUG_JANK) {
                    Log.d(TAG, "Frame time appears to be going backwards.  May be due to a "
                            + "previously skipped frame.  Waiting for next vsync.");
                }
                scheduleVsyncLocked();
                return;
            }

            if (mFPSDivisor > 1) {
                long timeSinceVsync = frameTimeNanos - mLastFrameTimeNanos;
                if (timeSinceVsync < (mFrameIntervalNanos * mFPSDivisor) && timeSinceVsync > 0) {
                    scheduleVsyncLocked();
                    return;
                }
            }

            mFrameInfo.setVsync(intendedFrameTimeNanos, frameTimeNanos);
            mFrameScheduled = false;
            mLastFrameTimeNanos = frameTimeNanos;
        }

        try {
            Trace.traceBegin(Trace.TRACE_TAG_VIEW, "Choreographer#doFrame");
            AnimationUtils.lockAnimationClock(frameTimeNanos / TimeUtils.NANOS_PER_MS);

            mFrameInfo.markInputHandlingStart();
            doCallbacks(Choreographer.CALLBACK_INPUT, frameTimeNanos);

            mFrameInfo.markAnimationsStart();
            doCallbacks(Choreographer.CALLBACK_ANIMATION, frameTimeNanos);

            mFrameInfo.markPerformTraversalsStart();
            doCallbacks(Choreographer.CALLBACK_TRAVERSAL, frameTimeNanos);

            doCallbacks(Choreographer.CALLBACK_COMMIT, frameTimeNanos);
        } finally {
            AnimationUtils.unlockAnimationClock();
            Trace.traceEnd(Trace.TRACE_TAG_VIEW);
        }

        if (DEBUG_FRAMES) {
            final long endNanos = System.nanoTime();
            Log.d(TAG, "Frame " + frame + ": Finished, took "
                    + (endNanos - startNanos) * 0.000001f + " ms, latency "
                    + (startNanos - frameTimeNanos) * 0.000001f + " ms.");
        }
    }
```

以上代码表是经过时间毫秒校验， 是否掉帧等情况，会执行scheduleVsyncLocked();总之条件满足通过之后，执行doCallbacks方法：

```java
    void doCallbacks(int callbackType, long frameTimeNanos) {
        ......
			for (CallbackRecord c = callbacks; c != null; c = c.next) {
                if (DEBUG_FRAMES) {
                    Log.d(TAG, "RunCallback: type=" + callbackType
                            + ", action=" + c.action + ", token=" + c.token
                            + ", latencyMillis=" + (SystemClock.uptimeMillis() - c.dueTime));
                }
                c.run(frameTimeNanos);
            }
        .......
    }
```

取出之前CallbackRecord,即mTraversalRunnable作为action，执行其中的run方法，到此流程结束，后续会到ViewRootImpl中的doTraversal()，测量，布局，绘制了。

#### 卡顿分析

常见原因和解决方式

##### 过度绘制

* 去除不必要的背景色
* 布局的扁平化处理
* 减少透明色的使用

##### UI线程的复杂运算

* 减少UI线程中的数据运算

##### 频繁的GC

原因：内存抖动，瞬间产生大量的对象。

* 尽可能减少在for循环中new对象或在onDraw中创建对象等
* 尽量不要在循环中大量使用局部变量

