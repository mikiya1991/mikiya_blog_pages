---
title: "使用状态机进行协议处理"
layout: markdown
categories: 协议字段处理 状态机 代码复用 服务器
comments: True
---

最近在公司看前辈写的一个rtsp服务器的代码。获益良多。包括服务器的架构、具体处理思路都学到不少。

### 简单服务器架构

首先对于命令控制类型的服务器，比如rtsp。前辈使用的是多路转接+非阻塞的方式。

在具体的处理阶段，将处理过程分为了：接收数据、解析数据生成应答、发送应答、关闭死掉的会话。四个步骤，每个步骤都用状态机控制进度，比如接收数据阶段，有可能数据只到了一部分，没有全到，协议还无法解析，这时就将状态设置成“等待子数据”，这样下面的步骤就会主动忽略处理过程。

状态机的技巧无处不在，比如在协议处理过程中出错了。就将session的状态写成Terminate，这样应答时就会回复出错应答，并且关闭会话阶段会将其释放、关闭。

### 底层协议字段处理技巧

这段代码是一个使用for循环加状态机处理协议字段的示例。处理方式非常别具一格。

	if (peer->data_stat == RTSP_DATA_STAT_WAIT_REQUEST
		|| peer->data_stat == RTSP_DATA_STAT_WAIT_SUBSEQUENT_DATA) {

		int i;
		int stat = RTSP_DATA_STAT_AWAITING_DOLLAR;
		if (peer->recv[0] == '$') {
			stat = RTSP_DATA_STAT_AWAITING_STREAM_CHANNEL_ID;
			for (i = 1; i < len; i++) {
				if (stat == RTSP_DATA_STAT_AWAITING_STREAM_CHANNEL_ID)
					stat = RTSP_DATA_STAT_AWAITING_SIZE1;
				else if (stat == RTSP_DATA_STAT_AWAITING_SIZE1)
					stat = RTSP_DATA_STAT_AWAITING_SIZE2;
				else if (stat == RTSP_DATA_STAT_AWAITING_SIZE2)
					stat = RTSP_DATA_STAT_AWAITING_PACKET_DATA;
				else if (stat == RTSP_DATA_STAT_AWAITING_PACKET_DATA) {
					short size = 0;
					memcpy(&size, &peer->recv[i-2], 2);
					size = ntohs(size);
					if ((len - i) == size) {
						peer->data_stat = RTSP_DATA_STAT_PARSING_RTCP;
						return total;
					} else if ((len - i) > size) {
						int data_len = len-i-size;
						peer->data_stat = RTSP_DATA_STAT_WAIT_REQUEST;
						memcpy(peer->recv, &peer->recv[4+size], data_len);
						peer->recv_pos = &peer->recv[data_len];
						pe_log(RTSP, AV_LOG_INFO,
							"rtcp parse again %d, magic 0x%x\n",
							data_len, peer->recv[0]);
						if (peer->recv[0] == '$') {
							stat = RTSP_DATA_STAT_AWAITING_STREAM_CHANNEL_ID;
							len = data_len;
							i = 0;
						} else
							break;
					}
				}
			}
			if ((stat > RTSP_DATA_STAT_AWAITING_DOLLAR)
				&& (stat <= RTSP_DATA_STAT_AWAITING_PACKET_DATA)) {
				subseq = 1;
				pe_log(RTSP, AV_LOG_INFO,
					"rtcp wait subsequent data, stat %d\n", stat);
			}
		}
	}

平时处理时，都是按字节长度扫描。这段也是扫描，但是使用了状态机的方式，for加上状态变换，使得下面的代码区域自然成了一个状态处理区域。

相比较一个一个的顺序扫并处理，这段的处理更为优雅、漂亮。避免过长的代码，过多的if-else。有效的使代码复用。

不同字段相同的处理可以不用重复写。

是一种代码复用的写法，同时思路也更清晰。
