# ASUS STRIX RAID DLX — Windows 10 Fixes

[English](#english) · [Русский](#русский)

Community troubleshooting notes for audio stuttering, cold boot detection, the external control box and PotPlayer channel balance.

## Downloads / Скачать

| Package | Download |
| --- | --- |
| Cold boot detection / Холодный старт | [ASUS_STRIX_DLX_Cold_Start_Fix.zip](ASUS_STRIX_DLX_Cold_Start_Fix.zip?raw=true) |
| Control box startup / Автозапуск блока громкости | [ASUS_STRIX_DLX_Control_Box_Fix_v1.3_Clean.zip](ASUS_STRIX_DLX_Control_Box_Fix_v1.3_Clean.zip?raw=true) |
| PotPlayer stereo / Стерео в PotPlayer | [PotPlayer_ASUS_Stereo_Fix.zip](PotPlayer_ASUS_Stereo_Fix.zip?raw=true) |

The original archive instructions are in Russian; English troubleshooting steps are below. The full control box script v1.3 was tested on the owner's PC. The lighter Clean build has received static checks only.

Инструкции внутри исходных архивов — на русском языке. Полная версия скрипта блока громкости 1.3 проверена на компьютере владельца; облегчённая Clean прошла только статическую проверку.

## English

Hi everyone! I wanted to share fixes for four ASUS STRIX RAID DLX problems I ran into myself. Tracking them down took quite a while, so hopefully this saves another owner some time.

My current system is **Windows 10 Pro 22H2, build 19045, with a Gigabyte Z690 GAMING X DDR4 motherboard**. I previously experienced the cold boot detection issue on a Huananzhi X99-TF and an ASUS P6T Deluxe V2 as well.

**1. Audio stuttering and brief dropouts**

This was the first issue I encountered. The fix surprised me: **this PCIe sound card needed a USB controller driver**.

The STRIX RAID DLX audio device operates through an onboard ASMedia USB controller. Its controller driver matters alongside the ASUS audio driver. Installing a suitable USB controller driver resolved the stuttering in my earlier setup.

The [ASUS support page](https://www.asus.com/supportonly/strix%20raid%20dlx/helpdesk_download/) actually has a separate **USB → Asmedia USB3.1 Driver** download, with the note “Improve latency.”

I did not record the exact driver version that fixed that early issue. This and the next fix come from different stages of troubleshooting: I later encountered a separate USB driver problem.

**2. Card missing after a cold boot, working after a restart**

After the PC had been shut down for a while, Windows would not detect the STRIX audio device. Restarting brought it back. Changing motherboards did not eliminate the problem.

Diagnostics showed that the ASMedia USB controller was present, but its ASUS audio device was missing. Restarting that controller restored the sound.

**The fix on my current system was to replace ASMedia driver 1.16.47.2 with the built-in Microsoft USB xHCI driver.**

Manual steps:

1. Open **Device Manager → Universal Serial Bus controllers**.
2. Identify the controller belonging to the sound card. In my case, its hardware ID contains `PCI\VEN_1B21&DEV_1142&SUBSYS_86A01043`.
3. Select **Update driver → Browse my computer for drivers → Let me pick from a list of available drivers**.
4. Keep **Show compatible hardware** enabled and select the compatible option whose provider is **Microsoft**. Its name may appear as “USB xHCI Compliant Host Controller.”
5. Check the driver provider and audio after installation. Restart if Windows requests it.

This change applies to the sound card’s USB controller. Match the hardware ID to identify the correct device. If Windows does not offer a compatible Microsoft option, these steps cannot be reproduced using its offered driver list.

There was no separate Microsoft driver to download: it was already included with Windows.

Sound returned immediately after the driver change. I then tested a boot after two hours powered off, and another after leaving the PC off overnight with AC power disconnected. The card worked on the first boot in both cases. Windows Fast Startup was enabled for the overnight test, and audio remained free of stuttering after the switch.

**3. Audio works, but the external control box does not adjust the volume**

In my case, this involved startup of the ASUS component `nhAsusStrixSvc32.exe`. A regular startup shortcut did not work reliably: the audio device took time to appear, and the ASUS component could exit shortly after launch.

A Task Scheduler startup workaround helped. The script waits for the audio output to become ready, activates StrixControl, and retries if the ASUS process is absent.

The activation command can be run in PowerShell after the card has appeared:

```powershell
& "C:\Program Files\ASUSTeKcomputer.Inc\nhAsusStrix\UserInterface\nhAsusStrixSvc32.exe" /start StrixControl
```

That path is from my driver installation.

The full version 1.3 script was tested after powering the PC off and on, including overnight downtime: the volume dial worked. A lighter **1.3 Clean** edition was then prepared without its own diagnostic logs or startup shortcut migration. The Clean build itself has only received static checks so far.

For Clean: install the ASUS driver and Sonic Studio, extract the archive, run `1_INSTALL.cmd` from your normal user account, then sign out and back into Windows. Allow about three minutes and physically check the dial. `2_UNINSTALL.cmd` disables the workaround.

**4. PotPlayer plays only through the left channel, while other players work normally**

Windows had saved these channel levels for the **PotPlayer session on the STRIX RAID DLX output**:

- Left: **100%**
- Right: **0%**

Both channels looked normal inside PotPlayer itself. [NirSoft SoundVolumeView](https://www.nirsoft.net/utils/sound_volume_view.html) helped identify and fix the problem.

Play audio through ASUS in PotPlayer using shared rather than exclusive mode. Find its **Application** entry in SoundVolumeView and inspect **Channels Percent**.

Setting both channels to 100% restored stereo. This was my PowerShell command, run from the folder containing `SoundVolumeView.exe`:

```powershell
.\SoundVolumeView.exe /SetVolumeChannels "2- STRIX RAID DLX\Application\PotPlayer" 100 100
```

Your device name, including the `2-` prefix, may differ. Use the **Command-Line Friendly ID** from the correct SoundVolumeView entry. Check the result with Windows mono audio disabled.

The following archives have been prepared to help reproduce the fixes:

- `ASUS_STRIX_DLX_Cold_Start_Fix.zip` — instructions, diagnostics, driver switching and restoration of the previous driver.
- `ASUS_STRIX_DLX_Control_Box_Fix_v1.3_Clean.zip` — the lighter control box startup workaround, with the testing status described above.
- `PotPlayer_ASUS_Stereo_Fix.zip` — channel balance fix, SoundVolumeView and a left/right channel test.

All results described here are from a **STRIX RAID DLX running Windows 10**. I have not tested these fixes on Windows 11 or on RAID PRO/Soar cards.

If you have the same symptoms, try the relevant section and report your result along with your Windows version and controller driver. That should make it easier for other owners to compare their setups.

## Русский

Всем привет! Хочу поделиться решениями четырёх проблем ASUS STRIX RAID DLX, с которыми столкнулся сам. На их поиск ушло немало времени — надеюсь, мой опыт поможет другим владельцам этой карты.

Текущая система: **Windows 10 Pro 22H2, сборка 19045, Gigabyte Z690 GAMING X DDR4**. Ранее проблема холодного старта встречалась у меня и на Huananzhi X99-TF, и на ASUS P6T Deluxe V2.

**1. Заикания и кратковременные пропадания звука**

С этой проблемой я столкнулся ещё в самом начале. Решение оказалось неожиданным: **PCIe-карте потребовался драйвер USB-контроллера**.

У STRIX RAID DLX аудиочасть работает через внутренний USB-контроллер ASMedia. Поэтому значение имеет не только аудиодрайвер ASUS, но и драйвер этого контроллера. Установка подходящего USB-драйвера помогла мне устранить заикания.

На [странице поддержки ASUS](https://www.asus.com/supportonly/strix%20raid%20dlx/helpdesk_download/) действительно есть отдельный раздел **USB → Asmedia USB3.1 Driver** с примечанием «Improve latency».

Точную версию, которая помогла тогда, я не зафиксировал. Этот ранний случай и следующее исправление относятся к разным этапам диагностики: позже у меня возникла другая проблема с USB-драйвером.

**2. После холодного включения карта не определяется, после перезагрузки работает**

После длительного выключения компьютера звука не было: Windows не видела аудиочасть STRIX. Перезагрузка возвращала карту. Замена материнской платы проблему не устранила.

При диагностике выяснилось, что USB-контроллер ASMedia присутствует, а подключённое к нему аудиоустройство ASUS отсутствует. Перезапуск контроллера возвращал звук.

**Рабочим решением для моей системы стал переход с ASMedia 1.16.47.2 на встроенный драйвер Microsoft USB xHCI.**

Ручной способ:

1. Открыть **Диспетчер устройств → Контроллеры USB**.
2. Найти контроллер самой звуковой карты. В моём случае его ИД оборудования содержит `PCI\VEN_1B21&DEV_1142&SUBSYS_86A01043`.
3. Выбрать **Обновить драйвер → Найти драйверы на этом компьютере → Выбрать драйвер из списка доступных**.
4. Оставить включённым показ только совместимых устройств и выбрать вариант с поставщиком **Microsoft**. Название может выглядеть как «USB xHCI-совместимый хост-контроллер».
5. После установки проверить поставщика драйвера и звук; выполнить перезагрузку, если её запросит Windows.

Меняется именно USB-контроллер звуковой карты — сверяйте ИД оборудования, чтобы выбрать нужное устройство. Если совместимого варианта Microsoft нет, этот способ нельзя воспроизвести через предложенный Windows список.

Отдельно скачивать Microsoft xHCI не потребовалось: он уже входит в Windows.

После смены драйвера звук появился сразу. Затем я проверил включение после двух часов простоя и после ночи с отключённым сетевым питанием 220 В. В обоих случаях карта заработала с первого запуска. Быстрый запуск Windows к ночному тесту был включён, заиканий звука после перехода не было.

**3. Звук есть, но внешний блок не регулирует громкость**

В моём случае проблема была связана с запуском компонента ASUS `nhAsusStrixSvc32.exe`. Обычный ярлык в автозагрузке не обеспечивал устойчивой работы: аудиоустройство появлялось с задержкой, а компонент мог завершиться вскоре после запуска.

Помог автоматический запуск через Планировщик заданий. Скрипт ждёт готовности аудиовыхода, активирует StrixControl и повторяет попытку, если процесс ASUS отсутствует.

Команда активации, которую можно выполнить в PowerShell после появления карты:

```powershell
& "C:\Program Files\ASUSTeKcomputer.Inc\nhAsusStrix\UserInterface\nhAsusStrixSvc32.exe" /start StrixControl
```

Путь относится к моей установке драйвера.

Полная версия скрипта 1.3 проверена после повторного включения компьютера и ночного простоя: колесо заработало. На её основе подготовлена облегчённая **1.3 Clean** без собственных логов и переноса ярлыков. Именно сборка Clean пока прошла только статическую проверку.

Для Clean: установить драйвер ASUS и Sonic Studio, распаковать архив, запустить `1_INSTALL.cmd` из обычной учётной записи пользователя и заново войти в Windows. Дать запуску около трёх минут и проверить само колесо. Для отключения автозапуска предусмотрен `2_UNINSTALL.cmd`.

**4. В PotPlayer звук только в левом канале, другие плееры работают нормально**

У меня Windows сохранила для сеанса **PotPlayer на выходе STRIX RAID DLX** уровни каналов:

- левый — **100%**;
- правый — **0%**.

При этом в самом PotPlayer оба канала выглядели нормально. Помогла утилита [NirSoft SoundVolumeView](https://www.nirsoft.net/utils/sound_volume_view.html).

Нужно включить воспроизведение в PotPlayer через ASUS в общем, не эксклюзивном режиме, найти его строку с типом **Application** в SoundVolumeView и посмотреть поле **Channels Percent**.

Установка обоих каналов в 100% восстановила стерео. Моя команда в PowerShell, открытом из папки с `SoundVolumeView.exe`:

```powershell
.\SoundVolumeView.exe /SetVolumeChannels "2- STRIX RAID DLX\Application\PotPlayer" 100 100
```

Имя устройства, включая префикс `2-`, может отличаться. Подставляйте **Command-Line Friendly ID** нужной строки из SoundVolumeView. Проверять результат следует при выключенном системном монозвуке Windows.

Для повторения исправлений подготовлены архивы:

- `ASUS_STRIX_DLX_Cold_Start_Fix.zip` — инструкция, проверка, переключение драйвера и восстановление предыдущего.
- `ASUS_STRIX_DLX_Control_Box_Fix_v1.3_Clean.zip` — облегчённый автозапуск внешнего блока с указанным выше статусом проверки.
- `PotPlayer_ASUS_Stereo_Fix.zip` — исправление баланса, SoundVolumeView и тест левого/правого каналов.

Все описанные результаты получены на **STRIX RAID DLX под Windows 10**. На Windows 11 и моделях RAID PRO/Soar я эти исправления не проверял.

Если столкнулись с такими же симптомами, попробуйте соответствующий пункт и напишите результат вместе с версией Windows и драйвером контроллера — так другим владельцам будет проще сравнить свои случаи.
