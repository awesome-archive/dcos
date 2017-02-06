## DC/OS集群卸载

如果DC/OS的集群安装采用了**GUI**或**CLI**方式，就可以通过下述命令将集群从所有节点上全部卸载。

执行下述命令时，如果Bootstrap节点也存在于集群中，需要先按照集群节点管理中的[删除节点操作](/dcos-install-nodes-management.md)把Bootstrap节点从集群中删除。

登录到Bootstrap节点，执行下面的命令：

```
sudo bash dcos_generate_config.sh --uninstall
```

命令执行后控制台输出类似下面的内容：

```
Running mesosphere/dcos-genconf docker with BUILD_DIR set to /home/centos/genconf
====> EXECUTING UNINSTALL
This will uninstall DC/OS on your cluster. You may need to manually remove /var/lib/zookeeper in some cases after this completes, please see our documentation for details. Are you ABSOLUTELY sure you want to proceed? [ (y)es/(n)o ]: yes
====> START uninstall_dcos
====> STAGE uninstall
====> STAGE uninstall
====> OUTPUT FOR uninstall_dcos
====> END uninstall_dcos with returncode: 0
====> SUMMARY FOR uninstall_dcos
2 out of 2 hosts successfully completed uninstall_dcos stage.
====> END OF SUMMARY FOR uninstall_dcos
```

如果上述步骤输出结果出现错误，则需要重试或参照[删除单个节点](/dcos-install-nodes-management.md)的操作逐个移除所有节点。

在1.8版本，完全清除DC/OS可能会碰到一些问题，具体请参考：[DCOS-250](https://dcosjira.atlassian.net/browse/DCOS-250) 和 [DCOS-192](https://dcosjira.atlassian.net/browse/DCOS-192)。
