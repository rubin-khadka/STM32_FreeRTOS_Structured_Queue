# STM32 FreeRTOS Queue with Structure Data

## Overview
This project demonstrates advanced inter-task communication using queues to pass complex data structures in an STM32 microcontroller with FreeRTOS. Instead of simple integers, this example shows how to safely pass pointers to dynamically allocated structures between tasks.

> This project uses native FreeRTOS APIs directly, not the CMSIS-RTOS wrapper.

## Project Description
The application creates three tasks with different priorities:
- Sender1_Task (Priority: 2) - Creates and sends structured data every 2000ms
- Sender2_Task (Priority: 2) - Creates and sends structured data every 2000ms
- Receiver_Task (Priority: 1) - Receives and processes structured data every 3000ms

A queue with capacity of 2 structure pointers is used to pass data between sender tasks and the receiver task. This demonstrates how FreeRTOS queues can handle complex data types through dynamic memory allocation.

## Key Concept: Passing Complex Data Structures
- Dynamic Memory Allocation: Senders use pvPortMalloc() to create structures
- Pointer Queue: Queue stores pointers to structures, not the structures themselves
- Memory Management: Receiver must free the memory after processing
- Multiple Producers: Two identical priority tasks create different data patterns

## Hardware Requirements
- STM32 development board (STM32F103C8T6 "Blue Pill")
- USB-to-UART converter (for viewing debug messages)
- UART connection (PA9/PA10 for USART1 on STM32F103)

## Pin Configuration
| Pin | Function | Description |
|-----|----------|-------------|
| PA9 | USART1 TX | Debug output from STM32 |
| PA10 | USART1 RX | (Reserved for future use) |

## Structure Definition
```
typedef struct
{
  char *str;           // String message
  int counter;         // Incrementing counter
  uint16_t large_value; // Large numeric value
} Structured_Struct_t;
```

## Task Priorities and Behavior
| Task | Priority | Send/Receive Rate | Data Pattern | Role |
|------|----------|-------------------|--------------|------|
| Sender1_Task | 2 | Every 2000ms | counter: 1,2,3... | Producer |
| | | | value: 1000,1100,1200... |
| | | | str: "Hello From Sender1" | 
| Sender2_Task | 2 | Every 2000ms | counter: 1,2,3... | Producer |
| | | | value: 2000,2200,2400... |
| | | | str: "Hello From Sender2" |
| Receiver_Task | 1 | Every 3000ms | Processes any received data | Consumer |

## Queue Configuration
- Queue Length: 2 elements
- Element Size: sizeof(Structured_Struct_t*) (4 bytes on 32-bit system)
- Total Size: 8 bytes (for pointers only)
- Actual Data: Stored in heap, not in queue
- Access Method: FIFO (First In, First Out)

## Memory Management Flow
```
Sender Task                    Queue                      Receiver Task
     |                           |                            |
     | pvPortMalloc()            |                            |
     | Create structure          |                            |
     | Fill with data            |                            |
     |                           |                            |
     | xQueueSend() -----------> | Store pointer              |
     |                           |                            |
     |                           | xQueueReceive() --------> |
     |                           |                            |
     |                           |                            | Process data
     |                           |                            |
     |                           |                            | vPortFree()
     |                           |                            | Free memory
```

## Key Concepts Demonstrated
1. Passing Complex Data Types
- Queues can pass any data type by value or by reference
- This example passes pointers to avoid copying large structures
- Structure contains multiple data types (char*, int, uint16_t)

2. Dynamic Memory Management
- pvPortMalloc(): Allocate memory from FreeRTOS heap
- vPortFree(): Properly deallocate memory after use
- Critical to avoid memory leaks in long-running systems

3. Producer-Consumer Pattern
- Two producers create data at same rate (2 seconds)
- One consumer processes at slower rate (3 seconds)
- Queue buffers data when consumer is busy

4. Memory Ownership Transfer
- Sender creates and owns memory initially
- Send operation transfers ownership to queue
- Receive operation transfers ownership to receiver
- Receiver must free memory (ownership responsibility)

## Expected UART Output
```
Structured queue Successfully Created

Entered Sender1 Task
about to send to the queue

Successfully sent to the queue
Leaving Sender1 Task

Entered Sender2 Task
about to send to the queue

Successfully sent to the queue
Leaving Sender2 Task

Entered Receiver Task
about to receive from the queue

Received from Queue:
Counter = 1
Large value = 1000
String = Hello From Sender1

Leaving Receiver Task

Entered Receiver Task
about to receive from the queue

Received from Queue:
Counter = 1
Large value = 2000
String = Hello From Sender2

Leaving Receiver Task

... (pattern continues)
```

## Queue States and Memory Implications
When Queue Has Space
- Senders allocate memory and send successfully
- Pointers stored in queue
- Memory remains allocated until receiver processes

When Queue is Full
- Senders block in xQueueSend()
- No new memory allocation happens
- Existing data waits in queue

When Queue is Empty
- Receiver blocks in xQueueReceive()
- No data to process
- All memory has been freed

## Critical Memory Management Rules
1. Every malloc must have a corresponding free
    - Sender allocates
    - Receiver frees
2. Never free memory while still in use
    - Queue protects pointers during transit
    - Only receiver has pointer after receive
3. Check allocation success
    - Production code should verify pvPortMalloc() != NULL
4. Use appropriate free function
    - vPortFree() for memory allocated with pvPortMalloc()
    - Never use standard free()

## Comparison: Value vs Pointer Queue
| Aspect | Passing by Value | Passing by Pointer |
|--------|------------------|--------------------|
| Queue element size | sizeof(structure) | sizeof(pointer) - 4 bytes |
| Memory usage | Larger (copies data) | Smaller (copies address) |
| Memory location | Queue internal | Heap allocated |
| Flexibility | Fixed structure | Can modify dynamically |
| Risk | Stack overflow | Memory leaks, dangling pointers |

## Contact
**Rubin Khadka Chhetri**  
📧 rubinkhadka84@gmail.com <br>
🐙 GitHub: https://github.com/rubin-khadka
