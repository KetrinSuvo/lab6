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

___________________________________________________________________________--
_______________________________________________________________________________----

Качаем исходники
which ps
dpkg -S /bin/ps
apt-get source procps

cd /procps-3.3.9/ps
нам нужен файл output.c

ketrin@ketrin-desktop:~/lab6$ grep "pr_pcpu" procps-3.3.9/ps/*
procps-3.3.9/ps/output.c:static int pr_pcpu(char *restrict const outbuf, const proc_t *restrict const pp){
procps-3.3.9/ps/output.c:{"%cpu",      "%CPU",    pr_pcpu,     sr_pcpu,    4,   0,    BSD, ET|RIGHT}, /*pcpu*/
procps-3.3.9/ps/output.c:{"pcpu",      "%CPU",    pr_pcpu,     sr_pcpu,    4,   0,    U98, ET|RIGHT}, /*%cpu*/

предварительно закоментировав строки с вызовами
пишем функцию

static int mypr_pcpu(char *restrict const outbuf, const proc_t *restrict const pp){
  unsigned long long start_cputime, end_cputime; 
  unsigned long long start_etime, end_etime; 
  float pcpu = 0; 

  unsigned long t;
  unsigned dd,hh,mm,ss;
  char *cp = outbuf;
  int c;

  t = cook_time(pp);
  ss = t%60;
  t /= 60;
  mm = t%60;
  t /= 60;
  hh = t%24;
  t /= 24;
  dd = t;
  
  start_cputime = dd * 3600 + mm * 60 + ss;
  
  t = cook_etime(pp);
  ss = t%60;
  t /= 60;
  mm = t%60;
  t /= 60;
  hh = t%24;
  t /= 24;
  dd = t;

  start_etime = dd * 3600 + mm * 60 + ss;

  sleep(1);

  t = cook_time(pp);
  ss = t%60;
  t /= 60;
  mm = t%60;
  t /= 60;
  hh = t%24;
  t /= 24;
  dd = t;
  
  end_cputime = dd * 3600 + mm * 60 + ss;
  
  t = cook_etime(pp);
  ss = t%60;
  t /= 60;
  mm = t%60;
  t /= 60;
  hh = t%24;
  t /= 24;
  dd = t;

  end_etime = dd * 3600 + mm * 60 + ss;

  pcpu =  ((end_cputime - start_cputime) / (end_etime - end_cputime)) * 100;
  return snprintf(outbuf, COLWID, "%g", pcpu);
}


В корне делаем make.
1. aclocal
2. autoconf
3. automake  --add-missing
4. ./configure
5. make

в итоге у нас соберется все, обращем внимание что в  каталоге ps получившаяся утилита называется не ps а pscommand.
проверяем вызов 


 dd if=/dev/zero of=/dev/null & sleep 3; kill -STOP $!; top -b -n1 -p$!; ./pscommand up $!;


потом перемещаемся в корень нашего каталога с исходниками, делаем
make clean
dpkg-source --commit

нас спросят про имя заплаты. вводим ps_CPU%_lab6_top)
и получаем нашу заплатку по следующему пути:
procps-3.3.9/debian/patches/ps_CPU%_lab6_top





