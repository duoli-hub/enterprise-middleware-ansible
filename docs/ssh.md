# 调试 ssh 免密登录（首次，加密）：单机nginx
```bash
# pip安装
yum install -y python3-pip
pip3 install ansible

# 拉取代码
dnf install git -y
git clone https://github.com/duoli-hub/enterprise-middleware-ansible.git
cd enterprise-middleware-ansible/
chmod +x deploy
./deploy --help

# 创建加密变量文件
echo "CloudEasy2020" > .vault_pass.txt
chmod 600 .vault_pass.txt

# 生成加密密码字符串并写入文件
ansible-vault encrypt_string '1' --name 'ansible_ssh_pass' --vault-password-file .vault_pass.txt --encrypt-vault-id default

cat > inventory/production/host_vars/nginx-lb-01.yml << 'EOF'
ansible_ssh_pass: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          31353736393562616237323565386538366135636261643132386434363432346439643562336666
          6439336565396331376464336235303235306231626637380a653930323934323730623139356632
          37306563636565366662616438663533373230616335396533623837663938303139666465386563
          3762633131656136300a303833636265343234656264306635663265363461316364643339626135
          6338
EOF

# 修复python解释器警告（可选）
cat >> inventory/production/host_vars/nginx-lb-01.yml << 'EOF'
ansible_python_interpreter: /usr/bin/python3.9
EOF

# 设置主机ip
vim inventory/production/hosts.ini
nginx-lb-01 ansible_host=192.168.174.136

# 执行免密配置(提示输入SSH密码，直接回车即可)
./deploy ssh --limit nginx-lb-01 --no-vault


# 验证免密是否生效：如果不提示密码并返回 免密成功，则一切正常。
ssh -o PasswordAuthentication=no 192.168.174.136 "echo '免密成功'"
```
---

# 调试 ssh 免密登录（首次，非加密）：单机nginx
```bash
# pip安装
yum install -y python3-pip
pip3 install ansible

# 拉取代码
dnf install git -y
git clone https://github.com/duoli-hub/enterprise-middleware-ansible.git
cd enterprise-middleware-ansible/
chmod +x deploy
./deploy --help

# 将root密码写入文件
cat > inventory/production/host_vars/nginx-lb-01.yml << 'EOF'
ansible_ssh_pass: "1"
EOF

# 修复python解释器警告（可选）
cat >> inventory/production/host_vars/nginx-lb-01.yml << 'EOF'
ansible_python_interpreter: /usr/bin/python3.9
EOF

# 设置主机ip
vim inventory/production/hosts.ini
nginx-lb-01 ansible_host=192.168.174.136

# 执行免密配置(提示输入SSH密码，直接回车即可)
./deploy ssh --limit nginx-lb-01 --no-vault


# 验证免密是否生效：如果不提示密码并返回 免密成功，则一切正常。
ssh -o PasswordAuthentication=no 192.168.174.136 "echo '免密成功'"
```