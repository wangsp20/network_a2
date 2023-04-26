# network_a2
Task
Implement reliable transport protocol using Selective Repeat in C by studying and extending an implmentation of Go Back N running on a network simulator.

Background
You've been hired by "Net Source", a company specialising in networking software to build Selective Repeat code for their network emulation tool. They have an existing implementation of Go-Back-N that interfaces with their emulator. Their programmer, Mue, who was working on an implementation of the Selective Repeat protocol has caught the flu and the implementation must absolutely be finished in the next week.  As the code isn't finished, Mue hasn't completed testing yet. Mue is an experienced C programmer and there is nothing wrong with the C syntax or structure of the code that Mue has written. So you don't have to correct any C syntax errors, your job is to finish the code, correct any protocol errors and test it. 

Although you are not required to use this code, they expect much of this code is reusable and only some modifications are needed to change Go-Back-N to Selective Repeat

system overview

There are two hosts (A and B). An application on host A is sending messages to an application on host B. The application messages (layer 5) on host A are sent to layer 4 (transport) on host A, which must implement the reliable transport protocol for reliable delivery. Layer 4 creates packets and sends them to the network (layer 3). The network transfers (unreliably) these packets to host B where they are handed to the transport layer ( receiver) and if not corrupt or out of order, the message is extracted and delivered to the receiving application (layer 5) on host B.

The file gbn.c has the code for the Go Back N sender procedures A_output(), A_init(), A_input(), and A_timerinterrupt(). gbn.c also has the code for the Go Back N receiver procedures B_input() and B_init(). At this stage, only unidirectional transfer of data (from A to B) is required, so B does not need to implement B_timerinterrupt(). Of course, B will have to send packets to A to acknowledge receipt of data.

Go Back N Software Interface
The routines are detailed below. Such procedures in real-life would be part of the operating system, and would be called by other procedures in the operating system. In the simulator the simulator will call and be called by procedures that emulate the network environment and operating system.

void A_output(struct msg message)
message is a structure containing data to be sent to B. This routine will be called whenever the upper layer application at the sending side (A) has a message to send.  It is the job of the reliable transport protocol to insure that the data in such a message is delivered in-order, and correctly, to the receiving side upper layer. 

void A_input(struct pkt packet)
This routine will be called whenever a packet sent from B (i.e., as a result of a tolayer3() being called by a B procedure) arrives at A. packet is the (possibly corrupted) packet sent from B.

void A_timerinterrupt(void)
This routine will be called when A's timer expires (thus generating a timer interrupt). This routine controls the retransmission of packets. See starttimer() and stoptimer()  below for how the timer is started and stopped.

void A_init(void)
This routine will be called once, before any other A-side routines are called. It is used to do any required initialization. 

void B_input(struct pkt packet)
This routine will be called whenever a packet sent from A (i.e., as a result of a  tolayer3() being called by a A-side procedure) arrives at B. The packet is the (possibly corrupted) packet sent from A. 

void B_init(void)
This routine will be called once, before any other B-side routines are called. It is used to do any required initialization.

The unit of data passed between the application layer and the transport layer protocol is a message, which is declared as: 

struct msg {
  char data[20];
};
That is, data is stored in a msg structure which contains an array of 20 chars. A char is one byte. The sending entity will thus receive data in 20-byte chunks from the sending application, and the receiving entity should deliver 20-byte chunks to the receiving application.

The unit of data passed between the transport layer and the network layer is a packet, which is declared as: 

struct pkt {
   int seqnum;
   int acknum;
   int checksum;
   char payload[20];
};
The A_output() routine fills in the payload field from the message data passed down from the Application layer. The other packet fields must be filled in the a_output routine so that they can be used by the reliable transport protocol to insure reliable delivery, as we've seen in class.

These functions implement what the sender and receiver should do when packets arrive.

The Emulator Software Interface
This section will describe the functions of the emulator code that you will be using in your implementation. Do not edit the emulator code.

The procedures described above implement the reliable transport layer protocol and are the procedures that you will be implementing.

The following emulator procedures can be called by the reliable transport procedures you write. They are explained here so you know how they fit in. They are not part of the reliable transport implementation and these routines work correctly.

void starttimer(int calling_entity, double);
Where calling_entity is either A (for starting the A-side timer) or B (for starting the B side timer), and increment is a float value indicating the amount of time that will pass before the timer interrupts. A's timer should only be started (or stopped) by A-side routines, and similarly for the B-side timer. To give you an idea of the appropriate increment value to use: a packet sent into the network takes an average of 5 time units to arrive at the other side when there are no other messages in the medium. You are free to experiment with different timeout values; but when handing in, the timeout value must be set to 16.0

Note that starttimer() is not the same as restarttimer(). If a timer is already running it must be stopped before it is started. Calling starttimer() when the timer is already running, or calling stoptimer() when the timer is not running, indicates an error in the protocol behaviour and will result in an error message.

The emulator code is in the file emulator.c.Use the emulator header file to refere to the definitions of the emulator functions. You do not need to understand or look at emulator.c; but if you want to know how the emulator works, you are welcome to look at the code.

The starttimer() call should occur immediately after the tolayer3() call that sends the packet being timed.

void stoptimer(int calling_entity)
Where calling_entity is either A (for stopping the A-side timer) or B (for stopping the B side timer).

void tolayer3(int calling_entity, struct packet)
Where calling_entity is either A (for the A-side send) or B (for the B side send). Calling this routine will cause a the packet to be sent into the network, destined for the other entity.

void tolayer5(int calling_entity, char[20] message)
Where calling_entity is either A (for A-side delivery to layer 5) or B (for B-side delivery to layer 5). With unidirectional data transfer, you would only be calling this when calling_entity is equal to B (delivery to the B-side). Calling this routine will cause data to be passed up to layer 5.
