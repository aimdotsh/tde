TDE 与 ADG 测试

采用rman duplicate 进行搭建ADG，与常规的ADG相同，但是需要在 duplicate 之前，配置好tde，并将钱包打开。

ADG 建议配置自动登录钱包。



```
ADMINISTER KEY MANAGEMENT CREATE  AUTO_LOGIN KEYSTORE FROM KEYSTORE '/etc/ORACLE/WALLETS/tdecdb' IDENTIFIED BY pwd200;
```

