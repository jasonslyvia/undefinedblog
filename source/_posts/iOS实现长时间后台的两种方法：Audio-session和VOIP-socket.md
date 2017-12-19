---
title: iOS实现长时间后台的两种方法：Audio session和VOIP socket
tags: 

  - ios
permalink: ios-background-running-methods
id: 121
updated: '2012-12-04 09:31:08'
date: 2012-12-04 09:31:08
---

<p>我们知道 iOS 开启后台任务后可以获得最多 600 秒的执行时间，而一些需要在后台下载或者与服务器保持连接的 App 是如何突破 600 秒的限制的呢？像网易公开课就可以在后台持续下载，优酷也可以在后台持续缓存，这是怎么做到的呢？一般来说，要实现 iOS 长时间后台运行，需要声明 VOIP、Audio 或 GPS。</p>
<h2>Audio session</h2>
<p>实现方法很简单，就是在后台一直播放一个无声的音乐文件，这样就相当于声明了 Audio，就可以轻松突破 600 秒的限制了。</p>
<p>通过播放&ldquo;静默&rdquo;音让程序在后台执行的做法（即在 audio unit 回调函数中使用&nbsp;kAudioUnitRenderAction_OutputIsSilence 标志位），虽然确实可以实现后台执行，但实践中限制很多。最大的问题就在于程序的 audio session 不能被打断。当程序执行在后台时，只要另一个程序使用 kAudioSessionCategory_RecordAndPlay （比如 Skype）或者 kAudioSessionCategory_SoloAmbientSound（印象中使用这个session 的不多），那么本程序就会被立即打断。</p>
<p>打断本身不是问题，但当播放程序被打断时，唯一能够获得的只有处理 audio session interruption 的很短一段时间。我的实验测试大概是 3 到 5 秒，但因为程序随后立即暂停，无法挂调试器，所以很难准确估计，然后程序就会被立即转入休眠状态。这点时间和 applicationDidEnterBackground 回调函数所用的时间大体相当，但是因为这个打断中间还伴随一个播音的回调动作，程序结构不是很好组织，在很多情况下是不够做现场保存工作的。</p>
<p>不推荐播放&ldquo;静默&rdquo;音的另一个问题它的恢复播放需要的场景非常麻烦。比如当一个 iOS 程序在后台被 VOIP 唤醒时，它是不能直接获得 audio unit 重新开始播音的。如果此时调用 AudioOutputUnitStart() ，会返回一个错误码，哪怕前台没有任何一个程序在运行也是如此。此时你不可能让程序重新进入稳定运行状态。有些没经验的程序员喜欢利用 audio session interruption handler 做所谓的自动播放恢复，但他们其实都没有注意到 audio session interruption 的状态恢复回调并不会保证被调用。目前实测能自动恢复调用的，大概也就只有内建的电话拨号程序，以及一些非常特殊的场景（比如你用一个 MP3 播放器打断 audio session，然后杀掉 MP3 播放器进程，然后把被打断的程序重新置回前台）。这样经常导致的结果，就是你开心地发现程序没问题，然后在放进生产环境中发现各种时不时的崩溃或启动失败。</p>
<h2>VOIP socket</h2>
<p>VOIP &nbsp;Socket 可以在后台运行。当程序进入后台时，事实上整个程序被暂停运行，但 VOIP socket 因为受系统控制而不在此列。我的观察是，每次有新的数据来临时，程序会被唤醒并执行大约几秒钟，然后再次进入休眠。Stackoverflow 上的说法是 10 秒钟，但我不确定，可能是我的试验不够精确。</p>
<p>参考资料：<a href="http://www.zhihu.com/question/20634271" target="_blank">知乎</a> &nbsp;<a href="http://stackoverflow.com/questions/6601536/ios-voip-socket-will-not-run-in-background" target="_blank">StackOverFlow</a>&nbsp;&nbsp;<a href="http://developer.apple.com/library/ios/#documentation/Audio/Conceptual/AudioSessionProgrammingGuide/AudioSessionCategories/AudioSessionCategories.html" target="_blank">AppleDevCenter</a></p>
