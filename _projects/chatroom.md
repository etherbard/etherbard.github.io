---
title: "Chatroom Based on Multicast"
excerpt: "<img src='/images/crawler/image-20210723213938332.png' width='500'>"
collection: projects
---

## 1 Requirements

Write a chatroom program based on multicast using Socket programming to enable text and file transmission.

## 2 Experiment Environment

3 Ubuntu 16.04 virtual machines.

![vm](/images/chatroom/vm.jpg)

## 3 Modules

### 3.1 Main Body

The main body of the program consists of `Chat.java`, `GUI.java`, `Config.java`, and `Logger.java`.

`Chat.java` is the interface of the program, controlling other modules.

`GUI.java` is the graphic interface of the program.

`Config.java` is the configuration of the program.

`Logger.java` outputs program running information to the console and log files.

### 3.2 Command Processing

The command processing module consists of `CommandListener.java`, `CommandHandler.java`, `Message.java`, `Integrity.java`, and `RetransmissionHandler.java`.

`CommandListener.java` declares the functions of the program, including sending and receiving character messages, sending files, logging in and out, etc.

`CommandHandler.java` is the core of the entire program and is responsible for implementing most of the program's functions. It mainly includes joining multicast groups, sending and receiving character messages through multicast, sending and receiving files, message integrity verification, requesting retransmission, etc. The running thread is responsible for receiving messages, verifying the integrity of the received messages, and judging whether any messages are lost during transmission through the message sequence maintained by the program to make a retransmission request. For file transfer requests, the file sender will first publish a message through multicast. The message starts with `FILE` and contains the name of the transferred file. When the file receiver receives this message, it will immediately call the file receiving function, writing the received bit stream to the file created by the previously obtained file name, and then resume the message listening mode.

`Message.java` processes messages, which mainly includes converting sent messages into bit streams, converting received bit streams into messages in a specific format, and processing retransmission request messages.

`Integrity.java` is the integrity verification of messages. During the message sending phase, the program calculates the hash value of the program and sends it out via multicast along with the text. When a message is received, the program calculates the hash value of the message and compares it with the received hash value. If there is an inconsistency, a retransmission is requested. This program uses SHA-1 as the digest algorithm.

`Retransmission.java` implements the message retransmission mechanism. During multicast transmission, messages may be lost. This program implements message retransmission by maintaining a local historical message record. The program maintains a thread for receiving retransmission requests. When a message retransmission request is received, the program searches the historical message list for the message number that needs to be retransmitted, and transmits it again.

### 3.3 Data Structures

The data structure module consists of `Message.java`, `Payload.java`, `Request.java`, and `MessageSequence.java`.

`Message.java` declares the format and method of the transmitted message.

`Payload.java` declares the payload format and method of UDP messages.

`MessageSequence.java` declares the format and methods of maintained message sequences.

`Request.java` declares the format and method of requesting retransmission messages.

## 4 Results

Compile and run the multicast chat program on three virtual machines respectively. The program interface is as follows。
\begin{figure}[H]
\centering
\includegraphics[width=0.4\textwidth]{logon.jpg}
  % \hspace{1in}
  \includegraphics[width=0.4\textwidth]{chat.jpg}
\caption{Login and Chatroom}
\label{fig:logon}
\end{figure}

Messages are sent from three virtual machines respectively, and the information can be successfully received on the other two virtual machines.
\begin{figure}[H]
\centering
\includegraphics[width=0.3\textwidth]{vmA.jpg}
\includegraphics[width=0.3\textwidth]{vmB.jpg}
\includegraphics[width=0.3\textwidth]{vmC.jpg}
\caption{Send and receive message}
\label{fig:message}
\end{figure}

Send the file test.txt on virtual machine A to the other two virtual machines and receive it successfully.
\begin{figure}[H]
\centering
\includegraphics[width=\textwidth]{sendfile.jpg}
\caption{Send a file}
\label{fig:sendfile}
\end{figure}

\begin{figure}[H]
\centering
\includegraphics[width=\textwidth]{receivefile1.jpg}
\includegraphics[width=\textwidth]{receivefile2.jpg}
\caption{Receive a file}
\label{fig:receivefile}
\end{figure}

Test the retransmission mechanism on virtual machine A and virtual machine B, and set the packet loss rate on virtual machine B to 0.5.
\begin{figure}[H]
\centering
\includegraphics[width=\textwidth]{missingprob.jpg}
\caption{Loss rate}
\label{fig:missingprob}
\end{figure}

Sending the numeric sequence 1,2,3... on virtual machine A and observing that virtual machine B loses message 3.
\begin{figure}[H]
\centering
\includegraphics[width=0.4\textwidth]{missA.jpg}
\includegraphics[width=0.4\textwidth]{missB.jpg}
\caption{Message loss}
\label{fig:miss}
\end{figure}

When virtual machine B successfully receives message 4, the program checks the message sequence and finds that message 3 is lost, so it sends a retransmission request to A. A receives B's retransmission request, retrieves historical messages, obtains the message with sequence number 3, and resends it to B.
\begin{figure}[H]
\centering
\includegraphics[width=\textwidth]{requestA.jpg}
\includegraphics[width=\textwidth]{requestB.jpg}
\caption{Retransmission}
\label{fig:retransmission}
\end{figure}

B successfully receives the retransmitted message 3.
\begin{figure}[H]
\centering
\includegraphics[width=\textwidth]{result.jpg}
\caption{Retransmission received}
\label{fig:result}
\end{figure}