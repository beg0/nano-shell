# Nano-Shell

> Not yet completed...

Nano-Shell is a light but powerful shell designed for embedded systems.

- with or without embedded operating system;
- main loop mode or interrupt mode;
- flash friendly: less than 2.5KB(build with arm-none-eabi-gcc 7.3.1 20180622, -O3);
- memory friendly: no malloc and free;
- highly configurable;
- command line editing, history record, auto complete, hot key bind, etc...


## 简介
TODO:

## 功能
TODO:

## 配置说明
TODO:

## 移植

### 1. first implement these functions([`@file shell_io.h`](/shell_io/shell_io.h)) in your project:
> this file may help: [`/shell_io/shell_io.c`](/shell_io/shell_io.c).
```c
/**
* @brief send a chararcter...
*
*/
extern void shell_putc(char ch);


/**
* @brief send string...
*
*/
extern void shell_puts(const char *str);


/**
 * @brief printf() for nano-shell
 *
 */
extern int shell_printf(const char *format, ...) __attribute__((format(printf, 1, 2)));


/**
 * @brief Get next character available from stream.
 *        NOT USED in <interrupt mode>.
 *
 * @param ch Return the character in `ch` if there was...
 * @return Result is non-zero if there was a character, or 0 if there wasn't.
 *
 * @note if you run nano-shell in <main loop mode>, to avoid losing characters, you'd better use a
 * low layer receive fifo. Take uart for example, you can detect incoming data using interrupts and
 * then store each received character in a first-in-first out (FIFO) buffer.
 *
 * then `shell_getc(char *ch)` may be like this:
 *
 * int shell_getc(char *ch)
 * {
 *   if (fifo_empty()) {  // if no character in fifo,
 *     return 0;          // return false
 *   }
 *
 *   *ch = fifo_pop();  // fifo is not empty, get it.
 *   return 1;          // return true
 * }
 *
 */
extern int shell_getc(char *ch);
```

### 2. then modify the configuration file: [`shell_config.h`](/shell_config.h)

### 3. three modes:

- without os, main loop mode:
  ```c
  #include "nano_shell.h"

  int main(void)
  {
    /* system init code... */

    /* nano-shell infinite loop. */
    nano_shell_loop(NULL);
  }
  ```

- without os, interrupt mode:
  ```c
  void your_interrupt_handler (void) // such as uart receive interrupt handler
  {
    /* your uart receive code */
    char ch = uart_get_char();

    /* nano-shell isr interface */
    nano_shell_run_in_isr(ch);
  }
  ```
  > recommended turn off some configurations in `shell_config.h` in this mode.

- with os, take freertos for example:
  ```c
  #include "nano_shell.h"

  int main(void)
  {
    /* system init code... */

    /* create nano_shell task */
    TaskHandle_t shellTaskHandle = NULL;
    xTaskCreate(nano_shell_loop, "shellTask", <stack_size>, NULL,
                <task_priority>, &shellTaskHandle);

    /* start rtos task scheduler */
    vTaskStartScheduler();
  }
  ```
  > When allocating the stack size for nano-shell, you should consider the memory occupied by commands added in nano-shell.

### 4. build, flash and try it.
