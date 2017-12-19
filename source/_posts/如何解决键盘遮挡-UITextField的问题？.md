---
title: 如何解决键盘遮挡 UITextField的问题？
permalink: ru-he-jie-jue-jian-pan-zhe-dang-uitextfieldde-wen-ti
id: 9
updated: '2011-10-17 19:16:13'
date: 2011-10-17 19:16:13
tags:
---

<p>可以将遮住的界面移动到。软键盘出来的时候就将界面移动，输入完成，移回到原来的位置<br />
[cc lang="c"]<br />
-(BOOL)textFieldShouldReturn:(id)sender<br />
{<br />
    self.view.center = CGPointMake(240,160);<br />
    [sender resignFirstResponder];<br />
    return YES;<br />
}<br />
- (BOOL)textFieldShouldBeginEditing:(id)sender<br />
{<br />
    if(sender==emailText)<br />
    {<br />
        self.center=CGPointMake(self.center.x,80);<br />
    }<br />
    else if(sender==speechText)<br />
    {<br />
        self.center=CGPointMake(self.center.x,0);<br />
    }<br />
    return YES;<br />
}<br />
[/cc]</p>
