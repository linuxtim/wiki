```
Квест по перепрошивке gk7205v300 + IMX335 + XM_XT25F128B
Бутлодер на пароле, flash залочена.

Скачайте и установите у себя Tftpd сервер: https://github.com/peacepenguin/tftpd64/releases
Скачайте и распакуйте в отдельную папку прошивку: https://github.com/OpenIPC/firmware/releases/download/latest/openipc.gk7205v300-br.tgz, в настройках Tftpd укажите путь к папке с прошивкой.
Найдите в чате OpenIPC Users (Ru) сообщение с тэгом #GkTool , установите на компьютер ToolPlatform-1.0.0-win32-x86_64.zip
Скачайте u-boot.bin.img_cut из чата OpenIPC Users (Ru)
Подключите камеру и компьютер через Ethernet к одному роутеру, чтобы компьютер и камера были в одной подсети.
Подключите к компьютеру USB-TTL 3.3V адаптер, рекомендуется FTDI232. Он встанет на какой-то COM-порт, посмотреть номер COM порта в Диспетчере задач
Подулючите RX/TX/GND на камере к TX/RX/GND на USB-TTL 3.3V адаптере
Скачайте и установите Putty. Выставить режим COM порта на скорость 115200. Зайти на камеру через Putty, по COM порту. Убедится,что провода подключены правильно, видно лог камеры и вводятся символы с клавиатуры.
ОБЯЗАТЕЛЬНО СОХРАНИТЕ МАС адрес камеры, в процессе прошивки он сотрётся.


1)В Putty: бэкап копированием на свой hls сервер из ос, перешитой купером:
mkdir /mnt/Public
mount -o nolock 192.168.1.15(тут указывается IP компьютера):/home/pi/nfs_share /mnt/Public
cat /dev/mtdblock0 > /mnt/Public/mtd0
cat /dev/mtdblock1 > /mnt/Public/mtd1
cat /dev/mtdblock2 > /mnt/Public/mtd2
cat /dev/mtdblock3 > /mnt/Public/mtd3
cat /dev/mtdblock4 > /mnt/Public/mtd4
выйти из программы Putty и закрыть её.

2) прошить xm бут u-boot.bin.img_cut через #gktool (ToolPlatform):
В ToolPlatform выбрать COM-порт, на котором висит адаптер, Transfer mode - Serial.
Во вкладке Burn Fastboot выбрать Flash type: spi nor, File: файл u-boot.bin.img_cut.
Отключить камеру, нажать кнопку Burn, выждать 5 секунд, включить камеру. Начнется прошивка.
Перезагрузить камеру по питанию

3) зайти в U-boot через PUTTY, нажимая несколько раз CTRL-C в момент включения камеры (очень быстро нажимать).
Ввести команды 
sf probe 0; sf lock 0;
sf erase 0 1000000

-----нажать Enter----

setenv soc gk7205v300
setenv osmem 32M
setenv totalmem 128M
saveenv
-----нажать Enter----


setenv gatewayip 192.168.1.1            //вводите ip адрес вашего шлюза/роутера
setenv ipaddr 192.168.1.14               //ip адрес камеры
setenv netmask 255.255.255.0            //Маска подсети
setenv serverip 192.168.1.15             //ip адрес компьютера на котором вы работаете и запущен TFTPD сервер
setenv ethaddr 05:68:31:be:da:38        //MAC вашей ip камеры обязательно 
saveenv 

-----нажать Enter----

setenv bootargs 'mem=${osmem:-32M} console=ttyAMA0,115200 panic=20 root=/dev/mtdblock3 rootfstype=squashfs init=/init mtdparts=sfc:256k(boot),64k(env),2048k(kernel),5120k(rootfs),-(rootfs_data)'
setenv bootcmd 'setenv setargs setenv bootargs ${bootargs}; run setargs; sf probe 0; sf read 0x42000000 0x50000 0x200000; bootm 0x42000000'
setenv uk 'mw.b 0x42000000 ff 1000000; tftp 0x42000000 uImage.${soc} && sf probe 0; sf erase 0x50000 0x200000; sf write 0x42000000 0x50000 ${filesize}'
setenv ur 'mw.b 0x42000000 ff 1000000; tftp 0x42000000 rootfs.squashfs.${soc} && sf probe 0; sf erase 0x250000 0x500000; sf write 0x42000000 0x250000 ${filesize}'
saveenv
-----нажать Enter----

run uk; run ur; reset
-----нажать Enter----

если перезагрузка не помогла, и в консоли проходят пробелы то повторите 2 шаг, после него камера работает.

после загрузки, выполнить firstboot в консоли Putty
```