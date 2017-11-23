win��д���룬linux������
---
Ŀ�꣺win��ʹ��IDEд�ô���󣬷���Ĵ��͵�linux�ϱ��룬����

# 1 ����
**linux**: ssh server  
**win**: scp  
scp �� git �� portable �汾���У��� `PortableGit\usr\bin\` ��  

# 2 �������
scp��ʽ����
```
remote_ip="192.168.x.x"
remote_dir="/home"
file_list="Makefile *.cpp *.h"

scp ${file_list} root@${remote_ip}:${remote_dir}
```

# 3 ִ��
���� ssh

```
usr="xxx"
remote_ip="192.168.x.x"
remote_dir=/home/$usr

CDIR="cd "${remote_dir}

cmd_r=${CDIR}
cmd_r=${cmd_r}"; ls *.cpp *.h" # ������Ҫִ�е�����
cmd_r=${cmd_r}"; make -j3"

ssh -t -t -p 22 root@${remote_ip} ${cmd_r}
# win��߻���ʾ���
# ����ű� sh, bash ������ִ�У�git portable �ﶼ��

```
���û�п��� RSA ������ᵯ���������  

���Ҫ���� ssh �� rsa��ȷ��`/etc/ssh/sshd_config`�е���������
```
RSAAuthentication yes
PubkeyAuthentication yes
AuthorizedKeysFile	%h/.ssh/authorized_keys # ȥ��ע��
```

�ѹ�Կ����ָ���û���ssh ��֤key�ĺ���(���ӵ��ļ�)  
֮��Ҫ���� ssh server
```
usr="xxx"

pub_key=".pub"
remote_dir="/"$usr"/.ssh/authorized_keys" # ȷ���� .ssh Ŀ¼

CDIR="cd "${remote_dir}
scp ${file_list} root@${remote_ip}:${remote_dir} # �����Ǹ���
```
win ��һ��Ҳ�������Ƶ�λ�÷���˽Կ