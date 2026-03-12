---
tags:
  - windows_server
  - RDP
author: '  - "https://winitpro.ru"'
excalidraw-export-transparentti: true
---

### Установка и активация сервера лицензирования RDS на Windows Server

Сервер **Remote Desktop Licensing** используется для выдачи и отслеживания клиентских терминальных лицензий RDS (**CAL**). Согласно схеме лицензирования Microsoft все пользователи, или устройства, подключающиеся к графическому сеансу рабочего стола RDS, должны быть лицензированы. В этой статье мы рассмотрим, как установить и активировать роль сервера лицензирования удаленных рабочих столов на хосте с Windows Server 2022, 2019 и 2016, а также как установить клиентские лицензии RDS CAL.

Содержание:

- [Установка роли Remote Desktop Licensing в Windows Server](https://winitpro.ru/index.php/2017/11/21/ustanovka-i-aktivaciya-rds-license-na-windows-server-2016/#h2_1)
- [Активация сервера лицензий RDS на Windows Server](https://winitpro.ru/index.php/2017/11/21/ustanovka-i-aktivaciya-rds-license-na-windows-server-2016/#h2_2)
- [Установка клиентских лицензий RDS CAL в Windows Server](https://winitpro.ru/index.php/2017/11/21/ustanovka-i-aktivaciya-rds-license-na-windows-server-2016/#h2_3)
- [Настройка серверов RDSH на использование сервера лицензий RDS](https://winitpro.ru/index.php/2017/11/21/ustanovka-i-aktivaciya-rds-license-na-windows-server-2016/#h2_4)
- [Управление клиентскими лицензиями CAL на RDS](https://winitpro.ru/index.php/2017/11/21/ustanovka-i-aktivaciya-rds-license-na-windows-server-2016/#h2_5)

### Установка роли Remote Desktop Licensing в Windows Server

Компонент Remote Desktop Licensing можно установить на любом хосте Windows Server (не обязательно устанавливать его на одном из серверов [фермы RDS](https://winitpro.ru/index.php/2022/02/17/ustanovka-nastrojka-remote-desktop-services-rds-windows-server/)). Если вы разворачиваете хост RD Licensing в домене AD, добавьте сервер во встроенную группу **Terminal Server License Servers** (иначе сервер не сможет выдать CAL типа RDS Per User пользователям домена).

Состав этой группы позволяют быстро найти все хосты с лицензиями RDS в домене.

![Доменная группа Terminal Server License Servers ](https://winitpro.ru/wp-content/uploads/2017/11/domennaya-gruppa-Terminal-Server-LicenseServers.jpg)

Установите службу Remote Desktop Licensing через консоль Server Manager (Add Roles and Features -> **Remote Desktop Services -> Remote Desktop Licensing**.

![Remote Desktop Licensing - служба лицензирования терминалов](https://winitpro.ru/wp-content/uploads/2017/11/remote-desktop-licensing-sluzhba-licenzirovaniya-t.png)

Дождитесь окончания установки роли.

![установка службы Remote Desktop Licensing](https://winitpro.ru/wp-content/uploads/2017/11/ustanovka-sluzhby-remote-desktop-licensing.png)

Также вы можете в [Windows Server установить компонент](https://winitpro.ru/index.php/2019/09/13/upravlenie-roles-features-windows-iz-powershell/) лицензирования RDS и инструменты управления RD Licensing с помощью PowerShell:  
`Install-WindowsFeature RDS-Licensing –IncludeAllSubFeature -IncludeManagementTools`

Выведите установленные компоненты RDS на сервере и проверьте, что RDS-Licensing и RDS-Licensing-UI установлены:

`Get-WindowsFeature -Name RDS* | Where installed`

![powershell проверить какие службы RDS установлены на Windows Server](https://winitpro.ru/wp-content/uploads/2017/11/powershell-proverit-ustanovlennie-sluzhby-rds.jpg)

Для управления службой RDS-Licensing используются две консоли:

- Remote Desktop Licensing Manager (`licmgr.exe`)
- RD Licensing Diagnoser (`lsdiag.msc`)

![Консоли управления сервером лицензирования RDS](https://winitpro.ru/wp-content/uploads/2017/11/consoli-upravleniya-rd-licensing.jpg)

## Активация сервера лицензий RDS на Windows Server

Чтобы сервер лицензирования RDS мог выдавать лицензии клиентам, его необходимо активировать. Откройте консоль **Remote Desktop Licensing Manager** (`licmgr.exe`), щелкните ПКМ по имени вашего сервера и выберите пункт меню **Activate Server**.

![активация сервера терминальных лицензий](https://winitpro.ru/wp-content/uploads/2017/11/aktivaciya-servera-terminalnyh-licenzij.png)

В мастере активации сервера лицензирования RDS нужно выбрать, хотите ли вы активировать сервер через интернет, с помощью браузера или по телефону.

![выберите метод активации сервера лицензирования](https://winitpro.ru/wp-content/uploads/2017/11/vyberite-metod-aktivacii-servera-licenzirovaniya.png)

Далее нужно будет заполнить ряд информации о вашей организации (часть полей является обязательными). ![информация об организации и компании](https://winitpro.ru/wp-content/uploads/2017/11/informaciya-ob-organizacii-i-kompanii.png)

Нажмите кнопку **Finish**. Должна появится надпись:

```
The license server has been successfully activated.
```

![сервер лицензирования RDS успешно активирован](https://winitpro.ru/wp-content/uploads/2017/11/server-licenzirovaniya-rds-uspeshno-aktivirovan.png)

Щелкните в консоли по имени сервера и выберите **Review Configuration**. В этом примере сервер лицензий RDS активирован и может выдавать лицензии клиентам в домене AD.

```
This license server is a member of the Terminal Server License Servers group in Active Directory. This license server will be able to issue RDS Per User CALs to users in the domain, and you will be able to track the usage of RDS Per User CALs.
```
```
This license server is registered as a service connection point (SCP) in Active Directory Domain Services.
```

![Review Configuration ](https://winitpro.ru/wp-content/uploads/2017/11/review-configuration.png)

## Установка клиентских лицензий RDS CAL в Windows Server

Теперь на сервер лицензирования нужно установить приобретенный вами пакет терминальных лицензий (RDS CAL, client access license). Есть два типа терминальных CAL:

- **На устройство (Per** **Device** **CAL****)** – лицензия назначается на устройство (компьютер). Дает право подключения к RDS серверам с одного устройства любому количеству пользователей. При первом подключении устройства к RDS ему назначается временная лицензия, а при втором – постоянная. Лицензия не являются конкурентными, это означает что если у вас 10 лицензий Per Device, то к вашему RDS серверу смогут подключится всего 10 компьютеров. Актуальная OVL лицензия называется так: `Win Remote Desktop Services CAL 2022 SLng OLV NL AP DCAL`
- **На пользователя (Per** **User** **CAL****)** – лицензия позволяет одному пользователю подключаться к RDS с любого количества компьютеров. Этот тип лицензии привязывается к учетной записи пользователя в Active Directory, но выдается не навсегда, а на срок от 52 до 89 дней (случайное число). Актуальная Open Value лицензия этого типа называется так: `Win Remote Desktop Services CAL 2022 SLng OLV NL AP UCAL`.
	**Если вы** [разворачиваете RDSH сервер в рабочей группе](https://winitpro.ru/index.php/2021/06/17/ustanovka-standalone-rdsh-windows-server-workgroup/) (не в домене), используйте лицензирование на устройство (Per Device RDS CAL). Иначе RDSH сервер будет каждые 60 минут завершать сеанс пользователей с сообщением: “ *Проблема с лицензией удаленных рабочих столов и ваш сеанс будет завершен через 60 мин /* *There is a problem with your Remote Desktop license, and your session will be disconnected in 60 minutes* ”. ![Проблема с лицензией удаленных рабочих столов и ваш сеанс будет завершен через 60 мин ](https://winitpro.ru/wp-content/uploads/2017/11/rds-licensing-issue-60-minut-seans.jpg)

Клиентские лицензия RDS которые вы используете должны быть совместимы с версией Windows Server, к которой подключается пользователь или устройство. Следующая таблица позволяет определить совместимость RDS CAL с версий Windows Server на сервере лицензирования RDS:

|  | **2008 R2 СAL** | **2012 CAL** | **2016 CAL** | **2019 CAL** | **2022 CAL** |
| --- | --- | --- | --- | --- | --- |
| 2008 R2 | Yes | No | No | No | No |
| 2012 | Yes | Yes | No | No | No |
| 2012 R2 | Yes | Yes | No | No | No |
| 2016 | Yes | Yes | Yes | No | No |
| 2019 | Yes | Yes | Yes | Yes | No |
| 2022 | Yes | Yes | Yes | Yes | Yes |

**Примечание**. RDS CAL для новых версий Windows Server нельзя установить на предыдущие версии WS. Например, **вы не сможете установить** 2022 RDS CAL на хост лицензирования Windows Server 2016. При попытке установить новые лицензии на старую версию Windows Server появится ошибка:

![rds ошибка неверный код при добавлении лицензий RDS CAL](https://winitpro.ru/wp-content/uploads/2017/11/rds-code-nevernaya-licensiya.jpg)

```
RD Licensing Manager
The license code is not recognized. Ensure that you have entered the correct license code.
```

В консоли Remote Desktop Licensing Manager щелкните по серверу и выберите **Install Licenses**.

![Установка терминальных CAL на RDS сервере](https://winitpro.ru/wp-content/uploads/2017/11/ustanovka-terminalnyh-cal-na-rds-servere.png)

Выберите способ активации (автоматически, через веб или по телефону) и программу лицензирования (в нашем случае Enterprise Agreement).

В сеть утекло уже довольно много enterprise agreement номеров для RDS (*4965437*). Найти номера думаю, не составит проблемы. Обычно даже не нужно искать кряки или активаторы.

![программа лицензирования Enterprise Agreement](https://winitpro.ru/wp-content/uploads/2017/11/programma-licenzirovaniya-enterprise-agreement.png)

Следующие шаги мастера зависят от того, какой тип лицензирования выбран. В случае Enterprise Agreement нужно указать его номер. Если выбран тип лицензирования License Pack (Retail Purchase), нужно будет указать 25-символьный ключ продукта, полученный от Microsoft или партнера. ![номер лицензионного соглашения](https://winitpro.ru/wp-content/uploads/2017/11/nomer-licenzionnogo-soglasheniya.png)

Укажите тип продукта (Windows Server 2022, 2019 или 2016), тип RDS CAL и количество терминальных лицензий, которые нужно установить на сервере.  
![тип и количество RDS лицензий](https://winitpro.ru/wp-content/uploads/2017/11/tip-i-kolichestvo-rds-licenzij.png)

Если нужно сконвертировать RDS лицензии User CAL в Device CAL (или наоборот), щелкните по пакету лицензий в консоли RD Licensing Manager и выберите Convert Licenses.

![сонвертировать rds cal из user в device](https://winitpro.ru/wp-content/uploads/2017/11/convert-rds-cal.jpg)

## Настройка серверов RDSH на использование сервера лицензий RDS

После установки роли RDSH на Windows Server пользователю могут использовать его в течении пробного (grace) периода 120 дней, после чего они не смогут подключиться к RDS. Чтобы ваши RDSH хосту могли получать CAL лицензии с RDS License сервера и выдавать их устройствам/пользователям, нужно указать адрес сервера с RDS лицензиями в настройках терминальных серверов RD Session Host.

Можно задать адрес сервера лицензирования в настройка коллекции на RDSH. Откройте Server Manager -> Remote Desktop Services -> Collections. В правом верхнем меню выберите **Tasks** -> **Edit Deployment Properties**.

![rds изменить настройки Edit Deployment Properties](https://winitpro.ru/wp-content/uploads/2017/11/rds-Edit-Deployment-Properties.jpg)

Перейдите на вкладку **RD Licensing**, выберите тип лицензирования (Per user или Per device в зависимости от имеющихся лицензий) и адрес сервера RDS. Нажмите Add -> Ok.

![rdsh изменить параметры лицензирования хоста](https://winitpro.ru/wp-content/uploads/2017/11/rdsh-parametry-licenzirovaniya.jpg)

Если не задать тип лицензирования, при подключении к RDS будет появляться предупреждение [Не задан режим лицензирования для сервера узла сеансов удаленных рабочих столов](https://winitpro.ru/index.php/2018/03/29/oshibka-ne-zadan-rezhim-licenzirovaniya-dlya-servera-uzla-seansov-rds/).

Можно задать настройки сервера лицензирования RDS через групповые политики. В домене нужно создать нужно [создать новую GPO в консоли GPMC](https://winitpro.ru/index.php/2022/11/15/gpmc-upravlenie-gruppovymi-politikami-active-directory/) и назначить ее на [OU](https://winitpro.ru/index.php/2021/11/15/sozdat-strukturu-ou-v-active-directory-powershell/) с RDS серверами (либо вы можете указать имя сервера лицензирования RDS с помощью [локального редактора групповых политик](https://winitpro.ru/index.php/2015/10/02/redaktor-gruppovyx-politik-dlya-windows-10-home-edition/) – `gpedit.msc`).

Перейдите в раздел Computer Configuration -> Policies -> Admin Templates -> Windows Components -> Remote Desktop Services -> Remote Desktop Session Host -> Licensing и настройте два параметра.

- **Use the specified Remote Desktop license servers** – укажите имя или IP адрес сервера лицензирования RDS;
- **Set the Remote Desktop licensing mode** – выбор тип клиентских лицензий (RDS CAL).

![GPO -> Remote Desktop Session Host -> Licensing](https://winitpro.ru/wp-content/uploads/2017/11/gpo-greater-remote-desktop-session-host-greater-licensing.png)

Если вы установили RDSH на ознакомительной редакции Windows Server Evaluation, нужно конвертировать его в полноценную версию согласно [инструкции](https://winitpro.ru/index.php/2017/04/10/upgrade-evaluation-windows-server-2016-edition/). Без конвертации службы RDSH на таком хосте будут работать только 120 дней даже после того, как вы нацелите его на активированный сервер лицензий RDS.

Также можно задать имя сервера лицензирования RDS и тип CAL с помощью PowerShell. Если у вас развернут [посредник RDS Connection Broker](https://winitpro.ru/index.php/2015/03/24/remote-desktop-services-2012-connection-broker-ha/), можно изменить настройки лицензирования с помощью команды:

`Set-RDLicenseConfiguration -LicenseServer @("rds-lic01.winitpro.loc") -Mode PerDevice -ConnectionBroker "rdcb01.winitpro.loc"`

Либо вы можете указать адрес сервера лицензирования и тип лицензий [в реестре с помощью PowerShell команд](https://winitpro.ru/index.php/2017/02/07/rabotaem-s-reestrom-windows-cherez-powershell/):

`# Тип лицензирования RDS 2 – Per Device CAL, 4 – Per User CAL   $RDSCALMode = 4   # Имя сервера лицензирования RDS   $RDSlicServer = "rds-lic01.winitpro.loc"   New-Item "HKLM:\SYSTEM\CurrentControlSet\Services\TermService\Parameters\LicenseServers"   New-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Services\TermService\Parameters\LicenseServers" -Name SpecifiedLicenseServers -Value $RDSlicServer -PropertyType "MultiString"   Set-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\RCM\Licensing Core\" -Name "LicensingMode" -Value $RDSCALMode`

Хосты RDSH для получения лицензий с сервера RDS Licensing обращаются к нему по следующим портам. Убедитесь, что они не блокируются межсетевыми экранами (или Windows Firewall):
- TCP/135 (Microsoft RPC)
- UDP/137 (NetBIOS Datagram Service)
- UDP/138 (NetBIOS Name Resolution)
- TCP/139 (NetBIOS Session Service)
- TCP/445 (SMB)
- TCP 49152 – 65535 — RPC динамический диапазон адресов

Вы можете проверить доступность портов с помощью утилиты [PortQry](https://winitpro.ru/index.php/2017/10/05/proverka-dostupnosti-tcp-i-udp-portov-s-pomoshhyu-portqryv2/) или командлета [Test-NetConnection](https://winitpro.ru/index.php/2016/09/08/tcp-port-ping-s-pomoshhyu-powershell/).

Запустите утилиту **Remote** **Desktop** **Licensing** **Diagnoser** (`lsdiag.msc`) на RDSH хосте и проверьте, что он видит сервер лицензирования и количество доступных RDS CAL.

```
RD Licensing Diagnoser did not identify any licensing problems for the Remote Desktop Session Host server.
```

![RD licensing diagnoser: проверить подключение к серверу лицензирования](https://winitpro.ru/wp-content/uploads/2017/11/rd-licensing-diagnoser-poroit-podkluchenie-k-serveru-licenziy.jpg)  
Если сервер сервера лицензий RDS не задан, или недоступен, в консоли Licensing Diagnoser будут присутствовать следующие предупреждения:

```
Licenses are not available for this Remoter Desktop Session Host server, and RD Licensing Diagnose has identified licensing problems for the RDSH
Number of licenses available for clients: 0
The licensing mode for the Remote Desktop Session Host server is not configured
Remote Desktop Session Host server is within its grace period, but the RD Session Host server has not been configured with any license server.
```

![rds diagnoser недоступны лицензии](https://winitpro.ru/wp-content/uploads/2017/11/rds-diagnoser-licenses-not-available.jpg)

**Примечание**. В нашем случае после указания нового сервера лицензирования, на RDP клиенте при подключении стала появляться ошибка `The remote session was disconnected because there are no Remote Desktop License Servers available to provide a license`. Решение – удаление ключа [L$RTMTIMEBOMB](https://winitpro.ru/index.php/2015/11/26/the-remote-desktop-session-host-server-does-not-have-a-remote-desktop-license-server-specified/) из реестра.

Можете подключиться к RDSH серверу с клиента и проверить, что сервер лицензирования назначил RDS CAL подключению. Откройте консоль Event Viewer и перейдите **Applications and Services Logs** -> **Microsoft** -> **Windows** -> **TerminalServices-Licensing** -> **Operational.** Событие успешной выдачи RDS CAL с Event ID 82 будет содержать такую надпись:

```
The "Temporary"  Windows Server 2022 : RDS Per Device CAL belonging to computer "DESKTOP-S6G9U9C" has been upgraded to "Permanent" Windows Server 2022 : RDS Per Device CAL.
```

![Логи выдачи RDS лицензий в Event Viewer](https://winitpro.ru/wp-content/uploads/2017/11/log-vydachy-rds-cal-licenziy.jpg)

## Управление клиентскими лицензиями CAL на RDS

Рассмотрим несколько типовых инструментов администратора при управлении RDS CAL на сервере лицензирования.  
Вы можете в консоли управления RD Licensing Manager отчет об использовании лицензий RDS CAL. Для этого в контекстном меню сервера выберите **Create** **Report** -> **CAL** **Usage**.

![сгенерирвать отчет об использовании лицензий rds cal](https://winitpro.ru/wp-content/uploads/2017/11/otchet-ob-ispolzovanii-rds-cal.jpg)

Вывести информацию по установленным и используемым лицензиям RDS CAL с помощью PowerShell:

`Get-WmiObject Win32_TSLicenseKeyPack|select-object KeyPackId,ProductVersion,TypeAndModel,AvailableLicenses,IssuedLicenses |ft`

![rdc licensing cal Win32_TSLicenseKeyPack](https://winitpro.ru/wp-content/uploads/2017/11/rdc-licensing-cal-Win32_TSLicenseKeyPack.jpg)

Если у вас закончились свободные лицензии, вы можете отозвать ранее выданные лицензии RDS Device CAL для неактивных компьютеров из консоли (правой кнопкой по лицензии и выберите **Revoke License**.

![RDS CAL отозвать лицензию](https://winitpro.ru/wp-content/uploads/2017/11/rds-cal-otozvat-licenziyu.jpg)

Также вы можете отозвать RDS CAL с помощью скрипта PowerShell:

`$RevokedPCName=”msk-pc2332”   $licensepacks = Get-WmiObject win32_tslicensekeypack | where {($_.keypacktype -ne 0) -and ($_.keypacktype -ne 4) -and ($_.keypacktype -ne 6)}   $licensepacks.TotalLicenses   $TSLicensesAssigned = gwmi win32_tsissuedlicense | where {$_.licensestatus -eq 2}   $RevokePC = $TSLicensesAssigned | ? sIssuedToComputer -EQ $RevokedPCName   $RevokePC.Revoke()`

Можно отозвать до 20% Per-Device RDS CALs. Per-User CALs отозвать нельзя.

---

[Предыдущая статья](https://winitpro.ru/index.php/2017/11/09/spisok-oshibok-aktivacii-windows-10-i-sposoby-ix-ispravleniya/) [Следующая статья](https://winitpro.ru/index.php/2017/11/22/excel-vba-macros-send-email-via-outlook/)

[Отказоустойчивый кластер Windows Server 2016 в рабочей группе (без домена)](https://winitpro.ru/index.php/2017/10/19/otkazoustojchivyj-klaster-windows-server-2016-v-rabochej-gruppe-bez-domena/ "Отказоустойчивый кластер Windows Server 2016 в рабочей группе (без домена) - В версиях Windows Server, предшествующих Windows Server 2016, создать отказоустойчивый кластер из...")

[PortQry — утилита для проверки доступности TCP/UDP портов из Windows](https://winitpro.ru/index.php/2017/10/05/proverka-dostupnosti-tcp-i-udp-portov-s-pomoshhyu-portqryv2/ "PortQry — утилита для проверки доступности TCP/UDP портов из Windows - Несмотря на то, что в Windows есть множество инструментов для диагностики проблем в TCP/IP сетях...")

[WSUS: Ручной импорт (добавление) обновлений из Microsoft Update Catalog](https://winitpro.ru/index.php/2017/09/21/wsus-ruchnoj-import-obnovlenij-iz-microsoft-update-catalog/ "WSUS: Ручной импорт (добавление) обновлений из Microsoft Update Catalog - Не все фиксы, исправления и обновления для продуктов Microsoft доступны для установки в консоли...")

[Просмотр и анализ логов RDP подключений в Windows](https://winitpro.ru/index.php/2018/09/25/analizing-rdp-logs-windows-terminal-rds/ "Просмотр и анализ логов RDP подключений в Windows")

[Конвертирование ознакомительной (Evaluation) редакции Windows Server в полную](https://winitpro.ru/index.php/2017/04/10/upgrade-evaluation-windows-server-2016-edition/ "Конвертирование ознакомительной (Evaluation) редакции Windows Server в полную")

[Исправить ошибку CredSSP шифрования Oracle при RDP подключении](https://winitpro.ru/index.php/2018/05/11/rdp-auth-oshibka-credssp-encryption-oracle-remediation/ "Исправить ошибку CredSSP шифрования Oracle при RDP подключении")

[Запуск программы без прав администратора и подавлением запроса UAC](https://winitpro.ru/index.php/2018/06/28/zapusk-programmy-bez-prav-admina-i-zaprosa-uac/ "Запуск программы без прав администратора и подавлением запроса UAC")

---

---

Комментариев: 75 [Оставить комментарий](https://winitpro.ru/index.php/2017/11/21/ustanovka-i-aktivaciya-rds-license-na-windows-server-2016/#submit)

---