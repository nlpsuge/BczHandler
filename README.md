# BczHandler
Extract data from zpk into Anki.

# Where to find *.zpk files
You may found *.zpk in you phone and then extract them by yourself. Or just download them from https://github.com/yanwt/baicizhan_zpk.

# How to use
1. Copy BczHandler.py into 'addons' folder
2. If you have cloned https://github.com/yanwt/baicizhan_zpk, browser in 'baicizhan_zpk-master' folder in terminal
3. Execute command </br>
for file in $(find . -name meta.json) ; do cat $file && echo  ;done > your_path/baicizhan_zpk-master/mata-json-content.txt
4. Open Anki, then modify 'your_path'
5. Create Field in Anki</br>
baicizhan-mean_cn</br>
baicizhan-mean_en</br>
reminder-baicizhan</br>
word_etyma</br>
6. Restart Anki
7. Browser your `Desk` in Anki, then click `Import selected word(s)...` in `BczHandler` menu. The shorcut is `Ctrl + C`.
