# ComfyUI: Установка

## 📥 Установка

ComfyUI: https://github.com/Comfy-Org/ComfyUI#manual-install-windows-linux

PyTorch: https://pytorch.org/get-started/locally/

Можно взять "portable" версию. Но лучше - руками:
- Обновить драйвер видюхи.
- Python 3.12 (читаем желательную версию питона в ReadMe от ComfyUI).
- Git.
  - Для не-технарей - лучше также поставить GUI вроде SourceTree (бесплатно) / SmartGit (платно).
- Склонировать репо ComfyUI через git.

Затем...

### Готовый скрипт (для видюх NVIDIA)
> [!NOTE]
> Вместо "portable" питона, мы сделаем нормальный `venv`, который в случае поломки чего-то всегда можно пересоздать, не ломая саму установленную ComfyUI со всеми нодами. При таком пересоздании может потребоваться выключить-перезапустить-включить кастомные ноды, чтобы они также до-поставили свои зависимости.

Под AMD/intel - разбираемся сами, по оф. инструкции выше 👆🏻.

Чтобы не парить себе мозг - для установки просто:
- создай текстовый файл в корне папки ComfyUI (что-то вроде `_venv_init.txt`),
- открой его текстовым редактором,
- вставь в него код из блока ниже,
- замени путь питона (`C:\Python\312\python.exe`) на тот, который у тебя в системе (но я вообще рекомендую сам питон всегда ставить в винду так же: по пути `C:\Python\<мажорная_версия_без_точки_одними_цифрами>`) и сохрани,
- переименуй файл, чтобы стал с расширением `.bat` (если делаешь через проводник - убедись, что у тебя расширения показываются),
- просто запусти и дождись завершения,
- не закрывая окна и ничего в нём не нажимая - проверь в конце, что нигде не написало ошибок.
- В самом конце он должен также написать, обнаружилась ли CUDA.

<details>
<summary>▶️ Скрипт на создание venv ◀️</summary>

```cmd
@echo off

:: Just to be safe - cd to the script's dir:
cd /d "%~dp0"

set "TORCH_INDEX=https://download.pytorch.org/whl/cu130"
:: Experimental/nightly - directly from Comfy's README, but looks outdated:
::set "TORCH_INDEX=https://download.pytorch.org/whl/cu132"

echo Creating venv...
"C:\Python\312\python.exe" -m venv venv

echo Activating venv...
call .\venv\Scripts\activate.bat

python.exe --version

echo.

echo Where python:
echo.
where python
echo.

echo Continue? Is the first listed python inside venv dir? (Enter - continue, Crl+C - exit)
pause

echo.
echo.
echo.

python.exe -m pip install -U pip

echo.

echo Installing torch: %TORCH_INDEX% ...
python.exe -m pip install torch torchvision torchaudio --index-url "%TORCH_INDEX%"

echo.
echo.
echo.

echo Installing ComfyUI requirements...
python.exe -m pip install -r requirements.txt
python.exe -m pip install -r manager_requirements.txt

echo.
echo.
echo.

echo Test CUDA support...
python.exe -c "import torch; print(f'torch: {torch.__version__}, CUDA: {torch.cuda.is_available()}, GPU: {torch.cuda.get_device_name(0) if torch.cuda.is_available() else \"N/A\"}')"

pause
```
</details>

### То же самое, но вручную копируя в `cmd` построчно

#### [venv](https://docs.python.org/3/library/venv.html)
```cmd
where python

cd "C:\path\to\ComfyUI\dir"
python -m venv ".\venv"
```

#### Установка зависимостей
Суть™ для NVIDIA:
```cmd
call ".\venv\Scripts\activate.bat"
where python

python -m pip install -U pip
python -m pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu130
python -m pip install -r requirements.txt
python -m pip install -r manager_requirements.txt
```

### `--index-url` vs `--extra-index-url` и CUDA 13.0 vs 13.2

TL;DR: правильно - так (конец 2026): `--index-url https://download.pytorch.org/whl/cu130`

<details>
<summary>▶️ Пояснение ◀️</summary>

> В [официальной инструкции от самих Comfy](https://github.com/Comfy-Org/ComfyUI#nvidia) рекомендуется ставить главную зависимость - PyTorch - такой командой:
> ```cmd
> pip install torch torchvision torchaudio --extra-index-url https://download.pytorch.org/whl/cu130
> ```
> 
> При этом, на [столь же официальном сайте PyTorch](https://pytorch.org/get-started/locally/) сказано ставить так:
> ```cmd
> pip install torch torchvision --index-url https://download.pytorch.org/whl/cu132
> ```
> 
> Судя по всему, команда Comfy скопировала этот шаблон давно - когда на сайте PyTorch также рекомендовалось ставить с аргументом `--extra-index-url` - и не обновляли с тех пор. Конкретно сейчас (ноябрь 2026) можно столкнуться с проблемами, если ставить так (скачается версия торча не оттуда, а с официального PyPI - и потому будет без поддержки CUDA, т.е. работать будет только на проце).
> 
> Касаемо же 13.0 vs 13.2 - сейчас PyTorch перешёл на 13.2 в качестве стабильной версии... однако, куча кастомных нод для ComfyUI была написана под 13.0, и включает в себя не только питоновский код, но и скомпилированные бинарники. Поэтому в нашем случае  -всё ещё предпочитаем 13.0, если не хотим экспериментировать с новой-понтовой *(и потенциально, более производительной на видюхах RTX 50xx)*, но нестабильной установкой.
</details>


### `bat` на обновление
Мы точно будем периодически обновлять Comfy. Делается это так:
- Закрыть Comfy (окно командной строки, не просто вкладку браузера);
- Скачать новую версию (поскольку у нас склонированно через гит - надо просто сделать git pull);
- 👉🏻 **Обновить зависимости** 👈🏻;
- Перезапустить Comfy.

Для главного шага с зависимостями - лучше тоже создать...

<details>
<summary>▶️ Готовый bat-файл на обновление ◀️</summary>

```cmd
@echo off

:: Just to be safe - cd to the script's dir:
cd /d "%~dp0"

call .\venv\Scripts\activate.bat

echo Where python:
echo.
where python
echo.

echo Continue? Is the first listed python inside venv dir? (Enter - continue, Crl+C - exit)
pause
echo.

python.exe -m pip install -U pip
python.exe -m pip install -r requirements.txt
python.exe -m pip install -r manager_requirements.txt
pause
```
</details>

### `bat` запуска
```cmd
@echo off

:: Just to be safe - cd to the script's dir:
cd /d "%~dp0"

call .\venv\Scripts\activate.bat

echo Where python:
where python
echo.

python.exe -s .\main.py --enable-manager
```

Чтобы [расшарить по локалке](https://comfyui-wiki.com/en/faq/how-to-access-comfyui-on-lan) - заменяем последнюю строку на:
```cmd
python.exe -s .\main.py --enable-manager --listen --port 8188
```

И потом с других компов открывать через: `http://<host-ip-address>:8188`

Для дебага (когда чо-то не работает и надо убедится, что сломалась сама Comfy, а не какие-то ноды):
```cmd
python.exe -s .\main.py --enable-manager --disable-all-custom-nodes
```
