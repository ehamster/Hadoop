```bash 
只下载不安装
pip download -d e:/mnt/ pandas
只安装
pip install --no-index --find-links="e:/mnt/" pandas
下载慢可以使用清华源
pip install pytorch-transformer== 1.1.0 -i https://pypi.tuna.tsinghua.edu.cn/simple
```

```bash
安装包和所有依赖
比如想安装requests
pip downolad -d /home/demo requests
观察下载顺序
Collecting requests
  Downloading https://files.pythonhosted.org/packages/51/bd/23c926cd341ea6b7dd0b2a00aba99ae0f828be89d72b2190f27c11d4b7fb/requests-2.22.0-py2.py3-none-any.whl (57kB)
    100% |████████████████████████████████| 61kB 69kB/s 
  Saved ./demo/requests-2.22.0-py2.py3-none-any.whl
Collecting urllib3!=1.25.0,!=1.25.1,<1.26,>=1.21.1 (from requests)
  Downloading https://files.pythonhosted.org/packages/e0/da/55f51ea951e1b7c63a579c09dd7db825bb730ec1fe9c0180fc77bfb31448/urllib3-1.25.6-py2.py3-none-any.whl (125kB)
    100% |████████████████████████████████| 133kB 85kB/s 
  Saved ./demo/urllib3-1.25.6-py2.py3-none-any.whl
Collecting certifi>=2017.4.17 (from requests)
  Downloading https://files.pythonhosted.org/packages/18/b0/8146a4f8dd402f60744fa380bc73ca47303cccf8b9190fd16a827281eac2/certifi-2019.9.11-py2.py3-none-any.whl (154kB)
    100% |████████████████████████████████| 163kB 21kB/s 
  Saved ./demo/certifi-2019.9.11-py2.py3-none-any.whl
Collecting chardet<3.1.0,>=3.0.2 (from requests)
  Downloading https://files.pythonhosted.org/packages/bc/a9/01ffebfb562e4274b6487b4bb1ddec7ca55ec7510b22e4c51f14098443b8/chardet-3.0.4-py2.py3-none-any.whl (133kB)
    100% |████████████████████████████████| 143kB 9.9kB/s 
  Saved ./demo/chardet-3.0.4-py2.py3-none-any.whl
Collecting idna<2.9,>=2.5 (from requests)
  Downloading https://files.pythonhosted.org/packages/14/2c/cd551d81dbe15200be1cf41cd03869a46fe7226e7450af7a6545bfc474c9/idna-2.8-py2.py3-none-any.whl (58kB)
    100% |████████████████████████████████| 61kB 25kB/s 
  Saved ./demo/idna-2.8-py2.py3-none-any.whl
Successfully downloaded requests urllib3 certifi chardet idna

创建requirements.txt
内容和刚刚下载顺序相反
idna-2.8-py2.py3-none-any.whl
chardet-3.0.4-py2.py3-none-any.whl
certifi-2019.9.11-py2.py3-none-any.whl
urllib3-1.25.6-py2.py3-none-any.whl
requests-2.22.0-py2.py3-none-any.whl

安装：sudo pip install -r requirements.txt
```
