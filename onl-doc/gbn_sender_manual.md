# GBN Sender Lab

## Overview

Go-Back-N ARQ is a specific instance of the automatic repeat request (ARQ) protocol, in which the sending process continues to send a number of frames specified by a window size even without receiving an acknowledgement (ACK) packet from the receiver. It is a special case of the general sliding window protocol with the transmit window size of N and receive window size of 1. It can transmit N frames to the peer before requiring an ACK. 

In this lab, you need to implement the sender part of GBN.

## Gettting Started

1. Download `gbn_skeleton.py` from OpenNetLab webpage

```python
class StudentGBNSender(GBNSender):
    async def setup(self):
        pass

    async def student_task(self, message):
        pass
```

you should implement the following functions in the skeleton code:

* `setup`: This functions is executed before using the `student_task` function to run each test case. You should setup data structure that is used in this lab.
* `student_task`: Main routine for sending message to receiver. The message is a string, you should encapsulate each character of the string into a packet. And send all the packets to the receiver using GBN protocol.

Besides, you could add your own functions to help you complete this lab.

2. Getting familiar with OpenNetLab API

variables you can use in this lab:

* time_out: max time(ms) before next retransimission
* window_size: GBN sender window size
* seqno_range: the seqno range from 1 to 2^n-1, where n is seqno bit width

3. Uplaod your code to online judge

## GBN sender pseudocode

```text
sender():
    upon Start:
        send winsize packets to destination
        add sent packets to buffer
        start timer

    upon Receiving Packet:
        if ackno is not valid:
            return
        remove the acked packets from buffer
        send remaining packets to destination (until buffer is full)
        add sent packets to buffer
        reset timer

    upon Timer Timeout:
        resend all the packets in buffer
        reset timer
```
