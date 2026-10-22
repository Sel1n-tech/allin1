Связка:
Windows 10/11
│
├── WSL2 + Ubuntu
│   └── основная Linux-среда для git, docker, kubectl, helm, заметок
├── Docker Desktop
│   └── Docker Engine + интеграция с WSL2
├── Multipass
│   └── быстрые Ubuntu-виртуалки для изучения VM, SSH, cloud-init, сетей
├── kind / k3d
│   └── локальный Kubernetes внутри Docker/WSL
└── GitHub-репозиторий
    └── личная книга, лабы, конфиги, заметки, чек-листы

Ориентировочный план ресурсов:
WSL2 Ubuntu:        8–12 GB RAM
Docker Desktop:     6–8 GB RAM
Multipass VM:       по 2 GB RAM на машину
Диск:               80–150 GB
Итого:              примерно 20–26 GB RAM

1. Для начала:
Проверим виртуализацию: 

вызываем powershell (Win + R) -> cmd: systeminfo | findstr /C:"Hyper-V"
Должно быть такое: Обнаружена низкоуровневая оболочка. Функции, необходимые для Hyper-V, отображены не будут.

или

Hyper-V Requirements:      VM Monitor Mode Extensions: Yes
                           Virtualization Enabled In Firmware: Yes
                           Second Level Address Translation: Yes
                           Data Execution Prevention Available: Yes

Если Virtualization Enabled In Firmware: No, нужно включить виртуализацию в BIOS/UEFI:
Intel VT-x / Intel Virtualization Technology
AMD-V / SVM Mode
После включения — перезагрузка.

2. Выполняй в PowerShell от имени администратора (Базовая подготовка)

2.1. Установи/обнови WSL2

wsl --install -d Ubuntu-24.04

Если Ubuntu-24.04 недоступна:

wsl --list --online
wsl --install -d Ubuntu

После установки перезагрузись.

Проверь: (powershell)
wsl -l -v

Должно быть примерно:
  NAME                   STATE           VERSION
* Ubuntu-24.04           Running         2

2.2. Включи Hyper-V, если нужна Multipass (все это дело в powershell)

Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All

Также пригодится:

Enable-WindowsOptionalFeature -Online -FeatureName VirtualMachinePlatform -All

После этого перезагрузись.
Проверь:

systeminfo | findstr /C:"Hyper-V"

2.3. Установи удобные инструменты (powershell)
Можно через winget:

winget install --id Microsoft.WindowsTerminal -e
winget install --id Microsoft.VisualStudioCode -e
winget install --id Docker.DockerDesktop -e
winget install --id Git.Git -e

Multipass: winget install --id Canonical.Multipass -e

Multipass — это простая программа от Canonical (компании, которая делает Ubuntu), которая позволяет одной командой поднимать лёгкие виртуальные машины с Ubuntu на твоём компьютере.
То есть ты находишься на Windows, запускаешь команду — и у тебя появляется отдельная Ubuntu-машина, в которую можно зайти, что-то ставить, ломать, чинить и удалять.
Для твоей задачи он хорош, потому что позволяет быстро делать “лабораторные Linux-машины”.
Например: multipass launch 24.04 --name node1 --cpus 2 --memory 2G --disk 10G
Этой командой ты создаёшь виртуальную машину node1 с Ubuntu 24.04, 2 CPU, 2 GB RAM и диском 10 GB.
Потом заходишь в неё: (powershell)
multipass shell node1
Смотришь список машин:
multipass list
Информация о машине:
multipass info node1
Останавливаешь:
multipass stop node1
Удаляешь:
multipass delete node1
multipass purge

2.4. Настрой ресурсы WSL2
Создай или отредактируй файл: notepad $env:USERPROFILE\.wslconfig

$env:USERPROFILE\.wslconfig — это файл настроек WSL2, который лежит в домашней папке твоего Windows-пользователя.
Он нужен, чтобы ограничить/настроить ресурсы виртуальной машины WSL2: RAM, CPU, swap и сетевые опции.
Почему именно $env:USERPROFILE
$env:USERPROFILE — это переменная PowerShell.
Она равна твоей Windows-папке пользователя, например: C:\Users\Ivan
То есть команда: notepad $env:USERPROFILE\.wslconfig
означает: открой блокнотом файл .wslconfig, который находится в папке текущего пользователя Windows.

Если ты запустишь: echo $env:USERPROFILE увидишь что-то вроде: C:\Users\ТвоёИмя

Файл .wslconfig должен лежать именно здесь, потому что WSL читает его глобально для своего виртуального окружения. 


Пример для 32 GB RAM: (.ini) 
[wsl2]
memory=12GB
processors=8
swap=4GB
localhostForwarding=true

Сохрани и перезапусти WSL: (powershell)
wsl --shutdown

Потом зайди в Ubuntu: (powershell)
wsl -d Ubuntu-24.04

2.5. Настрой Docker Desktop
Открой Docker Desktop:

Settings → Resources → WSL Integration

Включи интеграцию с твоим Ubuntu.
Также в: Settings → Resources → Advanced

можно выделить Docker, например: 
Memory: 8 GB
CPUs: 6–8

Проверь из WSL:
docker version
docker info
docker run --rm hello-world

Если docker не найден внутри WSL: (powershell)
wsl --shutdown

затем снова открой Ubuntu.



Шаг 1. Создаём виртуальную машину node1
Открой PowerShell и выполни эту команду (она скачает лёгкий образ Ubuntu 24.04 и создаст машину с 2 ядрами, 2 ГБ RAM и 10 ГБ диска): multipass launch 24.04 --name node1 --cpus 2 --memory 2G --disk 10G


Шаг 2. Проверяем, что машина создалась
Выполни команду, чтобы увидеть список всех машин:
multipass list

Ты должен увидеть примерно такой вывод:
Name                    State             IPv4             Image
node1                   Running           192.168.x.x      Ubuntu 24.04 LTS
(Статус должен быть Running, а не Stopped или Deleted).

Шаг 3. Заходим внутрь машины
Теперь, когда машина существует, команда, которую ты заметил, сработает идеально:
multipass shell node1





