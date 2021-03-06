---
layout: post
title: Работа с TRIM у SSD диска в Ubuntu
permalink: /rabota-s-trim-u-ssd-diska-v-ubuntu.html
categories: [ktulhu]
---


		
В рамках целевой программы апгрейда вычислительной техники, вместо верно прослужившено hp-compaq nc2400, в славной компании <a href="http://noteland.ru/">Ноутленд</a> был недорого приобретен hp-compaq 6910p. Необходимость апгрейда была обусловлена невысокой скоростью работы процессора U2500 установленного в nc2400. В качестве альтернативы апгрейда процессора рассматривался вариант замены жесткого диска на SSD накопитель. Но nc2400 имеет интерфейс жесткого диска IDE, а большинство SSD накопителей имеют интерфейс SATA. По совокупности вышеозначенных обстоятельств было решено произвести двойной апгрейд &#8212; ноутбука и сразу жесткого диска в нем.

<span id="more-275"></span>

В качестве SSD диска был выбран дешевый и сердитый вариант Intel X25-V объемом 40Gb. Проверенная марка производителя удачно сочетается с невысокой <a href="http://www.price-omsk.ru/Intel_ssd/">ценой этой модели диска в магазинах</a>. Относительно небольшой объем накопителя не является проблемой ибо основные объемные данные хранятся на внешних сетевых дисках и постоянно иметь их внутри ноутбука не имеет особого смысла.


Firmware диска была обновлена до <a href="http://downloadcenter.intel.com/Detail_Desc.aspx?agr=Y&#038;DwnldID=18363">последней версии</a>. На диск была установлена операционная система Ubuntu версии 10.10. Система без дополнительных усилий, сразу после установки, начала работать со всеми устройствами ноутбука Wi-Fi, 3G, Bluetooth&#8230;


SSD Диск Intel X25-V и операционная система Ubuntu 10.10 имеют поддержку <a href="http://ru.wikipedia.org/wiki/TRIM_%28%D0%BA%D0%BE%D0%BC%D0%B0%D0%BD%D0%B4%D0%B0_SSD%29">команды TRIM</a>:

<blockquote>
Команда TRIM была введена вскоре после появления SSD, чтобы сделать их конкурентоспособной альтернативой традиционным HDD в персональных компьютерах. Из-за того, что на внутреннем уровне реализация операций в SSD существенно отличаются от тех же операций в традиционных механических жёстких дисках, обычные методы ОС таких операций, как удаление файлов и форматирование диска (не обращаясь непосредственно к затрагиваемым секторам/страницам на накопителе), приводит к прогрессирующему ухудшению производительности операций записи на SSD. Применение TRIM позволяет устройству SSD уменьшить влияние сборки мусора, которая в противном случае в дальнейшем выразится падением производительности операций записи в затронутые секторы.
</blockquote>

Остается лишь задействовать поддержку этой технологии во благо SSD накопителя.

<h2>Проверить работает ли TRIM</h2>

Можно с помощью следующей последовательности команд исполняемых в консоли:


1. создаем временный файл заполненный случайными данными:

<pre>cd /</pre>
<pre>sudo dd if=/dev/urandom of=tmpfile count=10 bs=512k oflag=direct</pre>

2. находим номер сектора диска содержащего данные этого файла (нам нужно любое число из колонки begin_LBA):

<pre>sudo hdparm --fibmap tmpfile</pre>

3. проверяем содержимое этого сектора:

<pre>sudo hdparm --read-sector 17179648 /dev/sda</pre>

(вместо 17179648 подставьте номер сектора показанный командой из п.2 и вместо /dev/sda устройство которым обозначен SSD диск)

будет показан набор случайных данных похожий на:

<blockquote>

&#8230;

adb1 0d7a e027 042e 84ad 3231 2f6c 85f1

58ee feb6 cac0 d403 5df0 46db 2cfe 531c

3cbd 5a21 e0b6 9b57 799a 7ac8 dd15 3883

</blockquote>

4. удаляем файл:

<pre>sudo rm tmpfile</pre>
<pre>sudo sync</pre>

5. проверяем содержимое файла еще раз:

<pre>sudo hdparm --read-sector 17179648 /dev/sda</pre>

при включенной поддержке TRIM вывод будет содержать нули:

<blockquote>

&#8230;

0000 0000 0000 0000 0000 0000 0000 0000

0000 0000 0000 0000 0000 0000 0000 0000

0000 0000 0000 0000 0000 0000 0000 0000

</blockquote>

если вывод содержит не нули, а данные из п. 3, то поддержка TRIM не осуществляется и нужно:

<h2>Включить поддержку TRIM</h2>

Для включения поддержки TRIM нужно открыть для редактирования файл /etc/fstab например так:

<pre>sudo mc -e /etc/fstab</pre>

и заменить:

<blockquote>

UUID=b49c6a96-ee65-4c12-9aca-66f5ea927aab /               ext4     errors=remount-ro 0       1

</blockquote>

на:

<blockquote>

UUID=b49c6a96-ee65-4c12-9aca-66f5ea927aab /               ext4     discard,noatime,nodiratime,errors=remount-ro 0       1

</blockquote>

(UUID=&#8230; менять не нужно, он будет другим у вашего диска)

<strong>discard</strong> &#8212; включает поддержку TRIM.

<strong>noatime,nodiratime</strong> &#8212; отключает запись времени доступа к файлу, что полезно для SSD диска.


После редактирования /etc/fstab нужно перезагрузить систему и повторить тестирование поддержки TRIM как было описано ранее.

			<hr/>
Спасибо, включил trim.

