# lab6
звестно, что утилиты top(1) и ps(1) по-разному подсчитывают %CPU потребляемое процессорное время.

kotoff@ubuntu:~$ dd if=/dev/zero of=/dev/null & sleep 3; kill -STOP $!; top -b -n1 -p$!; ps up $!;
[1] 26671
top - 17:25:16 up 25 days,  7:32,  6 users,  load average: 0.29, 0.22, 0.23
Tasks:   1 total,   0 running,   0 sleeping,   1 stopped,   0 zombie
Cpu(s): 22.0%us,  7.2%sy,  1.4%ni, 67.5%id,  1.8%wa,  0.0%hi,  0.0%si,  0.0%st
Mem:   8192144k total,  5966876k used,  2225268k free,   359880k buffers
Swap:  4104188k total,   218332k used,  3885856k free,  3352336k cached
  PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND            
26671 kotoff    20   0  5604  588  500 T    0  0.0   0:02.99 dd                 

[1]+  Остановлено  dd if=/dev/zero of=/dev/null
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
kotoff   26671 99.6  0.0   5604   588 pts/1    T    17:25   0:02 dd if=/dev/zero


Необходимо разобраться самостоятельно (сложно) и понять почему так, или найти ответ в интернетах (просто).

Зачем "починить" утилиту ps, добавив "правильное" вычисление %CPU, на интервале sleep(3) в одну секунду.

Результат, как всегда представить на github.com в виде git-репозитария для сборки deb-пакета с изменениями в виде собственного патча.
