---
title: TCP协议实验 NetRiver平台 完整代码
permalink: tcpxie-yi-shi-yan-netriverping-tai-wan-zheng-dai-ma
id: 31
updated: '2011-12-07 18:57:22'
date: 2011-12-07 18:57:22
tags:
---

<p><strong>适用于北京工业大学《计算机网络》课程实验部分</strong></p>
<p>编程实现TCP协议收发</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<p>[cc lang="c++"]#include "sysinclude.h"</p>
<p>extern void tcp_DiscardPkt(char* pBuffer, int type);<br />
extern void tcp_sendReport(int type);<br />
extern void tcp_sendIpPkt(unsigned char* pData, UINT16 len, unsigned int srcAddr, unsigned int dstAddr, UINT8 ttl);<br />
extern int waitIpPacket(char *pBuffer, int timeout);<br />
extern unsigned int getIpv4Address();<br />
extern unsigned int getServerIpv4Address();</p>
<p>#define INPUT 0<br />
#define OUTPUT 1</p>
<p>#define NOT_READY 0<br />
#define READY 1</p>
<p>#define DATA_NOT_ACKED 0<br />
#define DATA_ACKED 1</p>
<p>#define NOT_USED 0<br />
#define USED 1</p>
<p>#define MAX_TCP_CONNECTIONS 5</p>
<p>#define INPUT_SEG 0<br />
#define OUTPUT_SEG 1</p>
<p>typedef int STATE;</p>
<p>enum TCP_STATES<br />
{<br />
CLOSED,<br />
SYN_SENT,<br />
ESTABLISHED,<br />
FIN_WAIT1,<br />
FIN_WAIT2,<br />
TIME_WAIT,<br />
};</p>
<p>int gLocalPort = 2007;<br />
int gRemotePort = 2006;<br />
int gSeqNum = 1234;<br />
int gAckNum = 0;</p>
<p>struct MyTcpSeg<br />
{<br />
unsigned short src_port;<br />
unsigned short dst_port;<br />
unsigned int seq_num;<br />
unsigned int ack_num;<br />
unsigned char hdr_len;<br />
unsigned char flags;<br />
unsigned short window_size;// ????????<br />
unsigned short checksum;//??<br />
unsigned short urg_ptr;<br />
unsigned char data[2048];//???<br />
unsigned short len;//????<br />
};</p>
<p>struct MyTCB<br />
{<br />
STATE current_state;<br />
unsigned int local_ip;<br />
unsigned short local_port;<br />
unsigned int remote_ip;<br />
unsigned short remote_port;<br />
unsigned int seq;<br />
unsigned int ack;<br />
unsigned char flags;<br />
int iotype;<br />
int is_used;<br />
int data_ack;<br />
unsigned char data[2048];<br />
unsigned short data_len;<br />
};</p>
<p>struct MyTCB gTCB[MAX_TCP_CONNECTIONS];//TCB??<br />
int initialized = NOT_READY;//?????????</p>
<p>int convert_tcp_hdr_ntoh(struct MyTcpSeg* pTcpSeg)//??????????????<br />
{<br />
if( pTcpSeg == NULL )<br />
{<br />
return -1;<br />
}</p>
<p>pTcpSeg-&gt;src_port = ntohs(pTcpSeg-&gt;src_port);<br />
pTcpSeg-&gt;dst_port = ntohs(pTcpSeg-&gt;dst_port);<br />
pTcpSeg-&gt;seq_num = ntohl(pTcpSeg-&gt;seq_num);<br />
pTcpSeg-&gt;ack_num = ntohl(pTcpSeg-&gt;ack_num);<br />
pTcpSeg-&gt;window_size = ntohs(pTcpSeg-&gt;window_size);<br />
pTcpSeg-&gt;checksum = ntohs(pTcpSeg-&gt;checksum);<br />
pTcpSeg-&gt;urg_ptr = ntohs(pTcpSeg-&gt;urg_ptr);</p>
<p>return 0;<br />
}</p>
<p>int convert_tcp_hdr_hton(struct MyTcpSeg* pTcpSeg)//??????????????<br />
{<br />
if( pTcpSeg == NULL )<br />
{<br />
return -1;<br />
}</p>
<p>pTcpSeg-&gt;src_port = htons(pTcpSeg-&gt;src_port);<br />
pTcpSeg-&gt;dst_port = htons(pTcpSeg-&gt;dst_port);<br />
pTcpSeg-&gt;seq_num = htonl(pTcpSeg-&gt;seq_num);<br />
pTcpSeg-&gt;ack_num = htonl(pTcpSeg-&gt;ack_num);<br />
pTcpSeg-&gt;window_size = htons(pTcpSeg-&gt;window_size);<br />
pTcpSeg-&gt;checksum = htons(pTcpSeg-&gt;checksum);<br />
pTcpSeg-&gt;urg_ptr = htons(pTcpSeg-&gt;urg_ptr);</p>
<p>return 0;<br />
}</p>
<p>unsigned short tcp_calc_checksum(struct MyTCB* pTcb, struct MyTcpSeg* pTcpSeg) //??TCP?checksum<br />
{<br />
int i = 0;<br />
int len = 0;<br />
unsigned int sum = 0;<br />
unsigned short* p = (unsigned short*)pTcpSeg;</p>
<p>if( pTcb == NULL || pTcpSeg == NULL )<br />
{<br />
return 0;<br />
}</p>
<p>for( i=0; ilen) &gt; 20 )<br />
{<br />
if( len % 2 == 1 )<br />
{<br />
pTcpSeg-&gt;data[len - 20] = 0;<br />
len++;<br />
}</p>
<p>for( i=10; ilocal_ip&gt;&gt;16)<br />
+ (unsigned short)(pTcb-&gt;local_ip&amp;0xffff)<br />
+ (unsigned short)(pTcb-&gt;remote_ip&gt;&gt;16)<br />
+ (unsigned short)(pTcb-&gt;remote_ip&amp;0xffff);<br />
sum = sum + 6 + pTcpSeg-&gt;len;<br />
sum = ( sum &amp; 0xFFFF ) + ( sum &gt;&gt; 16 );<br />
sum = ( sum &amp; 0xFFFF ) + ( sum &gt;&gt; 16 );</p>
<p>return (unsigned short)(~sum);<br />
}</p>
<p>int get_socket(unsigned short local_port, unsigned short remote_port)<br />
{<br />
int i = 1;<br />
int sockfd = -1;</p>
<p>for( i=1; isrc_port = pTcb-&gt;local_port;<br />
pTcpSeg-&gt;dst_port = pTcb-&gt;remote_port;<br />
pTcpSeg-&gt;seq_num = pTcb-&gt;seq;<br />
pTcpSeg-&gt;ack_num = pTcb-&gt;ack;<br />
pTcpSeg-&gt;hdr_len = (unsigned char)(0x50);<br />
pTcpSeg-&gt;flags = pTcb-&gt;flags;<br />
pTcpSeg-&gt;window_size = 1024;<br />
pTcpSeg-&gt;urg_ptr = 0;</p>
<p>if( datalen &gt; 0 &amp;&amp; pData != NULL )<br />
{<br />
memcpy(pTcpSeg-&gt;data, pData, datalen);<br />
}</p>
<p>pTcpSeg-&gt;len = 20 + datalen;</p>
<p>return 0;<br />
}</p>
<p>int tcp_kick(struct MyTCB* pTcb, struct MyTcpSeg* pTcpSeg)<br />
{<br />
pTcpSeg-&gt;checksum = tcp_calc_checksum(pTcb, pTcpSeg);</p>
<p>convert_tcp_hdr_hton(pTcpSeg);</p>
<p>tcp_sendIpPkt((unsigned char*)pTcpSeg, pTcpSeg-&gt;len, pTcb-&gt;local_ip, pTcb-&gt;remote_ip, 255);</p>
<p>//memcpy(&amp;(pTcb-&gt;last_seg), pTcpSeg, pTcpSeg-&gt;len);</p>
<p>if( (pTcb-&gt;flags &amp; 0x0f) == 0x00 ) ///////////////////////////////////////////////////发送了一个data报文 同时seq+=n<br />
{<br />
//data<br />
pTcb-&gt;seq += pTcpSeg-&gt;len - 20;<br />
}<br />
else if( (pTcb-&gt;flags &amp; 0x0f) == 0x02 )<br />
{<br />
//syn<br />
pTcb-&gt;seq++;<br />
}<br />
else if( (pTcb-&gt;flags &amp; 0x0f) == 0x01 )<br />
{<br />
//fin<br />
pTcb-&gt;seq++;<br />
}<br />
else if( (pTcb-&gt;flags &amp; 0x3f) == 0x10 )<br />
{<br />
//ack<br />
}</p>
<p>return 0;<br />
}</p>
<p>int tcp_closed(struct MyTCB* pTcb, struct MyTcpSeg* pTcpSeg)<br />
{<br />
if( pTcb == NULL || pTcpSeg == NULL )<br />
{<br />
return -1;<br />
}</p>
<p>if( pTcb-&gt;iotype != OUTPUT )<br />
{<br />
//to do: discard packet</p>
<p>return -1;<br />
}</p>
<p>pTcb-&gt;current_state = SYN_SENT;<br />
pTcb-&gt;seq = pTcpSeg-&gt;seq_num ;</p>
<p>tcp_kick( pTcb, pTcpSeg );</p>
<p>return 0;<br />
}</p>
<p>int tcp_syn_sent(struct MyTCB* pTcb, struct MyTcpSeg* pTcpSeg)<br />
{<br />
struct MyTcpSeg my_seg;</p>
<p>if( pTcb == NULL || pTcpSeg == NULL )<br />
{<br />
return -1;<br />
}</p>
<p>if( pTcb-&gt;iotype != INPUT )<br />
{<br />
return -1;<br />
}</p>
<p>if( (pTcpSeg-&gt;flags &amp; 0x3f) != 0x12 )<br />
{<br />
//to do: discard packet</p>
<p>return -1;<br />
}</p>
<p>pTcb-&gt;ack = pTcpSeg-&gt;seq_num + 1;<br />
pTcb-&gt;flags = 0x10;</p>
<p>tcp_construct_segment( &amp;my_seg, pTcb, 0, NULL );<br />
tcp_kick( pTcb, &amp;my_seg );</p>
<p>pTcb-&gt;current_state = ESTABLISHED;</p>
<p>return 0;<br />
}</p>
<p>int tcp_established(struct MyTCB* pTcb, struct MyTcpSeg* pTcpSeg)<br />
{<br />
struct MyTcpSeg my_seg;</p>
<p>if( pTcb == NULL || pTcpSeg == NULL )<br />
{<br />
return -1;<br />
}</p>
<p>if( pTcb-&gt;iotype == INPUT )<br />
{<br />
if( pTcpSeg-&gt;seq_num != pTcb-&gt;ack )<br />
{<br />
tcp_DiscardPkt((char*)pTcpSeg, STUD_TCP_TEST_SEQNO_ERROR);<br />
//to do: discard packet</p>
<p>return -1;<br />
}</p>
<p>if( (pTcpSeg-&gt;flags &amp; 0x3f) == 0x10 )<br />
{<br />
//get packet and ack it<br />
memcpy(pTcb-&gt;data, pTcpSeg-&gt;data, pTcpSeg-&gt;len - 20);<br />
pTcb-&gt;data_len = pTcpSeg-&gt;len - 20;</p>
<p>if( pTcb-&gt;data_len == 0 )<br />
{<br />
//pTcb-&gt;ack = pTcpSeg-&gt;seq_num + 1;<br />
//pTcb-&gt;ack++;<br />
}<br />
else<br />
{<br />
pTcb-&gt;ack += pTcb-&gt;data_len;<br />
pTcb-&gt;flags = 0x10;<br />
tcp_construct_segment(&amp;my_seg, pTcb, 0, NULL);<br />
tcp_kick(pTcb, &amp;my_seg);<br />
}<br />
}<br />
}<br />
else<br />
{<br />
if( (pTcpSeg-&gt;flags &amp; 0x0F) == 0x01 )<br />
{<br />
pTcb-&gt;current_state = FIN_WAIT1;<br />
}</p>
<p>tcp_kick( pTcb, pTcpSeg );<br />
}</p>
<p>return 0;<br />
}</p>
<p>int tcp_finwait_1(struct MyTCB* pTcb, struct MyTcpSeg* pTcpSeg)<br />
{<br />
if( pTcb == NULL || pTcpSeg == NULL )<br />
{<br />
return -1;<br />
}</p>
<p>if( pTcb-&gt;iotype != INPUT )<br />
{<br />
return -1;<br />
}</p>
<p>if( pTcpSeg-&gt;seq_num != pTcb-&gt;ack )<br />
{<br />
tcp_DiscardPkt((char*)pTcpSeg, STUD_TCP_TEST_SEQNO_ERROR);</p>
<p>return -1;<br />
}</p>
<p>if( (pTcpSeg-&gt;flags &amp; 0x3f) == 0x10 &amp;&amp; pTcpSeg-&gt;ack_num == pTcb-&gt;seq )<br />
{<br />
pTcb-&gt;current_state = FIN_WAIT2;<br />
//pTcb-&gt;ack++;<br />
}</p>
<p>return 0;<br />
}</p>
<p>int tcp_finwait_2(struct MyTCB* pTcb, struct MyTcpSeg* pTcpSeg)<br />
{<br />
struct MyTcpSeg my_seg;</p>
<p>if( pTcb == NULL || pTcpSeg == NULL )<br />
{<br />
return -1;<br />
}</p>
<p>if( pTcb-&gt;iotype != INPUT )<br />
{<br />
return -1;<br />
}</p>
<p>if( pTcpSeg-&gt;seq_num != pTcb-&gt;ack )<br />
{<br />
tcp_DiscardPkt((char*)pTcpSeg, STUD_TCP_TEST_SEQNO_ERROR);</p>
<p>return -1;<br />
}</p>
<p>if( (pTcpSeg-&gt;flags &amp; 0x0f) == 0x01 )<br />
{<br />
pTcb-&gt;ack++;<br />
pTcb-&gt;flags = 0x10;</p>
<p>tcp_construct_segment( &amp;my_seg, pTcb, 0, NULL );<br />
tcp_kick( pTcb, &amp;my_seg );<br />
pTcb-&gt;current_state = CLOSED;<br />
}<br />
else<br />
{<br />
//to do<br />
}</p>
<p>return 0;<br />
}</p>
<p>int tcp_time_wait(struct MyTCB* pTcb, struct MyTcpSeg* pTcpSeg)<br />
{<br />
pTcb-&gt;current_state = CLOSED;<br />
//to do</p>
<p>return 0;<br />
}</p>
<p>int tcp_check(struct MyTCB* pTcb, struct MyTcpSeg* pTcpSeg)<br />
{<br />
int i = 0;<br />
int len = 0;<br />
unsigned int sum = 0;<br />
unsigned short* p = (unsigned short*)pTcpSeg;<br />
unsigned short *pIp;<br />
unsigned int myip1 = pTcb-&gt;local_ip;<br />
unsigned int myip2 = pTcb-&gt;remote_ip;</p>
<p>if( pTcb == NULL || pTcpSeg == NULL )<br />
{<br />
return -1;<br />
}</p>
<p>for( i=0; ilen) &gt; 20 )<br />
{<br />
if( len % 2 == 1 )<br />
{<br />
pTcpSeg-&gt;data[len - 20] = 0;<br />
len++;<br />
}</p>
<p>for( i=10; i&gt;16)<br />
+ (unsigned short)(myip1&amp;0xffff)<br />
+ (unsigned short)(myip2&gt;&gt;16)<br />
+ (unsigned short)(myip2&amp;0xffff);<br />
sum = sum + 6 + pTcpSeg-&gt;len;</p>
<p>sum = ( sum &amp; 0xFFFF ) + ( sum &gt;&gt; 16 );<br />
sum = ( sum &amp; 0xFFFF ) + ( sum &gt;&gt; 16 );</p>
<p>if( (unsigned short)(~sum) != 0 )<br />
{<br />
// TODO:<br />
printf("check sum error!n");</p>
<p>return -1;<br />
//return 0;<br />
}<br />
else<br />
{<br />
return 0;<br />
}<br />
}</p>
<p>int tcp_switch(struct MyTCB* pTcb, struct MyTcpSeg* pTcpSeg)<br />
{<br />
int ret = 0;</p>
<p>printf("STATE: %dn", pTcb-&gt;current_state);</p>
<p>switch(pTcb-&gt;current_state)<br />
{<br />
case CLOSED:<br />
ret = tcp_closed(pTcb, pTcpSeg);<br />
break;<br />
case SYN_SENT:<br />
ret = tcp_syn_sent(pTcb, pTcpSeg);<br />
break;<br />
case ESTABLISHED:<br />
ret = tcp_established(pTcb, pTcpSeg);<br />
break;<br />
case FIN_WAIT1:<br />
ret = tcp_finwait_1(pTcb, pTcpSeg);<br />
break;<br />
case FIN_WAIT2:<br />
ret = tcp_finwait_2(pTcb, pTcpSeg);<br />
break;<br />
case TIME_WAIT:<br />
ret = tcp_time_wait(pTcb, pTcpSeg);<br />
break;<br />
default:<br />
ret = -1;<br />
break;<br />
}</p>
<p>return ret;<br />
}</p>
<p>int stud_tcp_input(char* pBuffer, unsigned short len, unsigned int srcAddr, unsigned int dstAddr)<br />
{<br />
//construct MyTcpSeg from buffer<br />
struct MyTcpSeg tcp_seg;<br />
int sockfd = -1;</p>
<p>//printf("Here len = %dn", len);</p>
<p>if( len &lt; 20 )<br />
{<br />
return -1;<br />
}</p>
<p>if(initialized == NOT_READY)<br />
{<br />
tcp_init(1);<br />
gTCB[1].remote_ip = getServerIpv4Address();<br />
gTCB[1].remote_port = gRemotePort;<br />
initialized = READY;<br />
}</p>
<p>memcpy(&amp;tcp_seg, pBuffer, len);</p>
<p>tcp_seg.len = len;</p>
<p>//convert bytes' order<br />
convert_tcp_hdr_ntoh(&amp;tcp_seg);</p>
<p>sockfd = get_socket(tcp_seg.dst_port, tcp_seg.src_port);</p>
<p>if( sockfd == -1 || gTCB[sockfd].local_ip != ntohl(dstAddr) || gTCB[sockfd].remote_ip != ntohl(srcAddr) )<br />
{<br />
printf("sock error in stud_tcp_input()n");<br />
return -1;<br />
}</p>
<p>//gTCB.local_ip = ntohl(dstAddr);<br />
//gTCB.remote_ip = ntohl(srcAddr);</p>
<p>//check TCP checksum<br />
if( tcp_check(&amp;gTCB[sockfd], &amp;tcp_seg) != 0 )<br />
{<br />
return -1;<br />
}</p>
<p>gTCB[sockfd].iotype = INPUT;<br />
memcpy(gTCB[sockfd].data,tcp_seg.data,len - 20);<br />
gTCB[sockfd].data_len = len - 20;</p>
<p>tcp_switch(&amp;gTCB[sockfd], &amp;tcp_seg);</p>
<p>return 0;<br />
}</p>
<p>void stud_tcp_output(char* pData, unsigned short len, unsigned char flag, unsigned short srcPort, unsigned short dstPort, unsigned int srcAddr, unsigned int dstAddr)<br />
{<br />
struct MyTcpSeg my_seg;<br />
int sockfd = -1;</p>
<p>if(initialized == NOT_READY)<br />
{<br />
tcp_init(1);<br />
gTCB[1].remote_ip = getServerIpv4Address();<br />
gTCB[1].remote_port = gRemotePort;<br />
initialized = READY;<br />
}</p>
<p>sockfd = get_socket(srcPort, dstPort);</p>
<p>if( sockfd == -1 || gTCB[sockfd].local_ip != srcAddr || gTCB[sockfd].remote_ip != dstAddr )<br />
{<br />
return;<br />
}</p>
<p>gTCB[sockfd].flags = flag;<br />
//gTCB.local_port = srcPort;<br />
//gTCB.local_ip = ntohl(srcAddr);<br />
//gTCB.remote_port = dstPort;<br />
//gTCB.remote_ip = ntohl(dstAddr);</p>
<p>tcp_construct_segment(&amp;my_seg, &amp;gTCB[sockfd], len, (unsigned char *)pData);</p>
<p>gTCB[sockfd].iotype = OUTPUT;</p>
<p>tcp_switch(&amp;gTCB[sockfd], &amp;my_seg);<br />
}</p>
<p>int stud_tcp_socket(int domain, int type, int protocol)<br />
{<br />
int i = 1;<br />
int sockfd = -1;</p>
<p>if( domain != AF_INET || type != SOCK_STREAM || protocol != IPPROTO_TCP )<br />
{<br />
return -1;<br />
}</p>
<p>for( i=1; isin_addr.s_addr);<br />
gTCB[sockfd].remote_port = ntohs(addr-&gt;sin_port);</p>
<p>stud_tcp_output( NULL, 0, 0x02, gTCB[sockfd].local_port, gTCB[sockfd].remote_port, gTCB[sockfd].local_ip, gTCB[sockfd].remote_ip );</p>
<p>len = waitIpPacket(buffer, 10);</p>
<p>if( len &lt; 20 )<br />
{<br />
return -1;<br />
}</p>
<p>if (stud_tcp_input(buffer, len, htonl(gTCB[sockfd].remote_ip), htonl(gTCB[sockfd].local_ip)) != 0){<br />
return 1;<br />
}<br />
else<br />
{<br />
return 0;<br />
}<br />
}</p>
<p>int stud_tcp_send(int sockfd, const unsigned char* pData, unsigned short datalen, int flags)<br />
{<br />
char buffer[2048];<br />
int len;</p>
<p>if( gTCB[sockfd].current_state != ESTABLISHED )<br />
{<br />
return -1;<br />
}</p>
<p>stud_tcp_output((char *)pData, datalen, flags, gTCB[sockfd].local_port, gTCB[sockfd].remote_port, gTCB[sockfd].local_ip, gTCB[sockfd].remote_ip);</p>
<p>len = waitIpPacket(buffer, 10);</p>
<p>if( len &lt; 20 )<br />
{<br />
return -1;<br />
}</p>
<p>stud_tcp_input(buffer, len, htonl(gTCB[sockfd].remote_ip), htonl(gTCB[sockfd].local_ip));</p>
<p>return 0;<br />
}</p>
<p>int stud_tcp_recv(int sockfd, unsigned char* pData, unsigned short datalen, int flags)<br />
{<br />
char buffer[2048];<br />
int len;</p>
<p>if( (len = waitIpPacket(buffer, 10)) &lt; 20 )<br />
{<br />
return -1;<br />
}</p>
<p>stud_tcp_input(buffer, len, htonl(gTCB[sockfd].remote_ip),htonl(gTCB[sockfd].local_ip));</p>
<p>memcpy(pData, gTCB[sockfd].data, gTCB[sockfd].data_len);</p>
<p>return gTCB[sockfd].data_len;<br />
}</p>
<p>int stud_tcp_close(int sockfd)<br />
{<br />
char buffer[2048];<br />
int len;</p>
<p>stud_tcp_output(NULL, 0, 0x11, gTCB[sockfd].local_port, gTCB[sockfd].remote_port, gTCB[sockfd].local_ip, gTCB[sockfd].remote_ip);</p>
<p>//recv ACK<br />
if( (len = waitIpPacket(buffer, 10)) &lt; 20 )<br />
{<br />
return -1;<br />
}</p>
<p>stud_tcp_input(buffer, len, htonl(gTCB[sockfd].remote_ip), htonl(gTCB[sockfd].local_ip));</p>
<p>//recv FIN<br />
if( (len = waitIpPacket(buffer, 10)) &lt; 20 )<br />
{<br />
return -1;<br />
}</p>
<p>stud_tcp_input(buffer, len, htonl(gTCB[sockfd].remote_ip), htonl(gTCB[sockfd].local_ip));</p>
<p>gTCB[sockfd].is_used = NOT_USED;</p>
<p>return 0;<br />
}<br />
[/cc]</p>
