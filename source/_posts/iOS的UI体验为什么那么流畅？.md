---
title: iOS的UI体验为什么那么流畅？
permalink: iosde-uiti-yan-wei-shi-yao-na-yao-liu-chang
id: 32
updated: '2011-12-09 07:52:43'
date: 2011-12-09 07:52:43
tags:
---

<p>虽然很多Android手机的配置都比iPhone要高，比如大多数Andorid手机的内存都有1GB，而iPhone 4S只有512MB内存，但用过iPhone的人都知道Android手机在使用的时候总感觉没有那么顺滑，究竟为什么会出现这种现象呢？一位软件工程师 和前Google实习生Andrew Munn<a href="https://plus.google.com/100838276097451809262/posts/VDkV9XaJRGS" target="_blank">解释说</a>是因为Android系统UI效率低下的框架设计的问题。</p>
<p>不过，这个实习生Andrew Munn是一个软件工程专业的本科毕业生，他在Android团队并没有在框架团队工作，也没有看过Android渲染的源代码，因此他所说的未必是 100%准确。并且他也曾经在Windows Phone团队工作过，因此可能会不自觉的对Android产生偏见。以下就是他对Android为什么没有iOS流畅体验的看法。</p>
<p>Android没有iOS流畅的原因并非Java GC导致暂停，也不是因为Android运行的是Java编译的bytecode而iOS运行的native code，根本的原因是，iOS的UI渲染采用实时优先级，而Android的UI渲染遵循传统电脑模式的主线程普通优先级。</p>
<p>这听起来似乎很抽象和难以理解，但大家可以尝试一下，使用你的iPad或者iPhone，打开Safari，然后加载一个复杂的网页，例如新浪网首 页，当网页加载到一半的时候，把你的手指放在屏幕上，并且四处移动，你会发现所有的渲染立刻停止，在你拿开手指前，网页永远也不会继续加载。</p>
<p>而在Android设备上重复这个操作，你会发现，浏览器会继续尝试加载页面并渲染HTML，试图多任务同时进行，因此对于Android来说，一个高效的双核处理器是很重要的，这也就是Galaxy S II能够非常平滑的原因。</p>
<p>在iOS中UI渲染过程具有绝对的优先等级，当用户接触到iPhone的触摸屏后，iOS中所有的进程都将停止，UI线程拦截了所有的事件，系统会 将所有资源用于渲染UI过程，以保证用户界面的实时渲染优先级。而在Android系统中UI渲染过程的优先级别却没有那么高，也就是说当你触摸 Android手机屏幕的时候，系统后台的程序并没有停止，仍然在继续运行之中，比如下载和查收短信，这样系统UI获得的资源就不够，这就是 Android系统不流畅的原因。</p>
<p>由于这个原因，新发布的Galaxy Nexus，甚至配备四核处理器的话说EeePad Transformer Prime平板电脑都无法保证顺滑的操作体验，这些设备只能与3年前的iPhone顺滑程度相比，那么Android团队为什么不从根本解决这个问题呢？</p>
<p>除了UI渲染之外，Android缺乏有效的的硬件加速也是一个原因，在不同的Android手机上的硬件加速存在巨大差异，而苹果是唯一一个既做硬件又做软件的手持设备公司，只有苹果可以在硬件中插入对软件的优化，使得基于苹果芯片的设备不仅省电，而且流畅。</p>
<p>实际上，Android的开发工作在第一代iPhone发布之前就已经开始了，原始Android原型体被设计成为使用键盘手机的设备，也就是黑莓 手机的竞争对手。UI渲染优先级别在有键盘的手机上并没有那么重要。但是在iPhone发布之后，Android小组为了快速推出能与iPhone竞争的 产品，迅速将Android改成触摸屏手机系统，但那时重写UI框架已经不可能了。因为如果这样Android应用市场中的所有程序将变得不可用，这种关 系将一直处于恶性循环之中。</p>
