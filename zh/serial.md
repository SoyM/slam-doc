# serial

串口

## 串口设置

```c++
// 打开串口 
open("/dev/ttyUSB0", O_RDWR | O_NOCTTY))
    
int set_opt(int fd, int nSpeed, int nBits, char nEvent, int nStop)
{
    struct termios newtio, oldtio;
    if (tcgetattr(fd, &oldtio) != 0)
    {
        perror("SetupSerial 1");
        return -1;
    }
    
    // 配置寄存器必须置0
    bzero(&newtio, sizeof(newtio));
    newtio.c_cflag |= CLOCAL | CREAD;
    newtio.c_cflag &= ~CSIZE;
    
    // 数据位
    switch (nBits)
    {
    case 7:
        newtio.c_cflag |= CS7;
        break;
    case 8:
        newtio.c_cflag |= CS8;
        break;
    }

    // 校检类型
    switch (nEvent)
    {
    // 奇校检
    case 'O':
        newtio.c_cflag |= PARENB;
        newtio.c_cflag |= PARODD;
        newtio.c_iflag |= (INPCK | ISTRIP);
        break;
    // 偶校检
    case 'E':
        newtio.c_iflag |= (INPCK | ISTRIP);
        newtio.c_cflag |= PARENB;
        newtio.c_cflag &= ~PARODD;
        break;
    case 'N':
        newtio.c_cflag &= ~PARENB;
        break;
    }
    
    // 设置波特率
    switch (nSpeed)
    {
    case 2400:
        cfsetispeed(&newtio, B2400);
        cfsetospeed(&newtio, B2400);
        break;
    case 4800:
        cfsetispeed(&newtio, B4800);
        cfsetospeed(&newtio, B4800);
        break;
    case 9600:
        cfsetispeed(&newtio, B9600);
        cfsetospeed(&newtio, B9600);
        break;
    case 115200:
        cfsetispeed(&newtio, B115200);
        cfsetospeed(&newtio, B115200);
        break;
    case 460800:
        cfsetispeed(&newtio, B460800);
        cfsetospeed(&newtio, B460800);
        break;
    default:
        cfsetispeed(&newtio, B9600);
        cfsetospeed(&newtio, B9600);
        break;
    }
    
    // 停止位
    if (nStop == 1)
        newtio.c_cflag &= ~CSTOPB;
    else if (nStop == 2)
        newtio.c_cflag |= CSTOPB;
    // 超时时间
    newtio.c_cc[VTIME] = 0;
    // 最小接收字符
    newtio.c_cc[VMIN] = 0;
    
    // 刷新输入和输出缓冲区
    tcflush(fd, TCIFLUSH);
    
    // 写入新配置
    if ((tcsetattr(fd, TCSANOW, &newtio)) != 0)
    {
        perror("com set error");
        return -1;
    }

    printf("串口设置完成!\n\r");
    return 0;
}
```

## 读取串口

```c++
// 固定 读取指定个数的数据 & ctrl + c 退出处理
ssize_t uart::safe_read(char *vptr, size_t n) {
  size_t nleft;
  ssize_t nread;
  char *ptr;

  ptr = vptr;
  nleft = n;

  while (nleft > 0) {
    if ((nread = read(_fd, ptr, nleft)) < 0) {
      // Interrupted system call
      if (errno == EINTR) {
        printf("(closed)(safe_read) uart");
        close(_fd);
        exit(-1);
      } else {
        printf("system errors!!");
        return -1;
      }
    } else if (nread == 0) {
      printf("can't read anything");
      break;
    }
    nleft -= nread;
    ptr += nread;
  }
  return (n - nleft);
}

ssize_t uart::uart_read(char *r_buf, size_t len) {
  ssize_t cnt = 0;
  fd_set rfds;
  struct timeval time;

  // ctrl+c exit
  if (errno == EINTR) {
    printf("(closed)(uart_read) uart");
    close(_fd);
    exit(-1);
  }

  /*将文件描述符加入读描述符集合*/
  FD_ZERO(&rfds);
  FD_SET(_fd, &rfds);

  time.tv_sec = 1;
  time.tv_usec = 0;

  /*实现串口的多路I/O*/
  ret = select(_fd + 1, &rfds, NULL, NULL, &time);

  switch (ret) {
  case -1:
    printf("(uart_read) select error!\n");
    if (errno == EINTR) {
      printf("(closed)(uart_read)");
      close(_fd);
      exit(-1);
    }
    return -1;

  case 0:
    printf("(uart_read) time over!\n");
    return -1;

  default:
    cnt = safe_read(r_buf, len);
    if (cnt == -1) {
      printf("(fail)(uart_read) \n");
      return -1;
    }
    return cnt;
  }
}
```

## 写入串口

```c++
ssize_t uart::safe_write(const char *vptr, size_t n) {
  size_t nleft;
  ssize_t nwritten;
  const char *ptr;

  ptr = vptr;
  nleft = n;

  while (nleft > 0) {
    if (errno == EINTR) {
      printf("(closed)(safe_write) uart");
      close(_fd);
      exit(-1);
    }
    if ((nwritten = write(_fd, ptr, nleft)) <= 0) {

      nleft -= nwritten;
      ptr += nwritten;
      return n;
    }
  }
}

ssize_t uart::uart_write(const char *w_buf, size_t len) {
  ssize_t cnt = 0;

  cnt = safe_write(w_buf, len);
  if (errno == EINTR) {
    printf("(uart_write) closed");
    close(_fd);
    exit(-1);
  }
  if (cnt == -1) {
    printf("(fail)uart write\n");
    return -1;
  }

  return cnt;
}
```

## 关闭串口

```c++
close(_fd);
```
