1.下載並進入資料夾
git clone https://github.com/circledog/ansible-ELK
cd ansible-elk

2. 安裝必要collection
ansible-galaxy install -r requirements.yml

3. 安裝k3s
ansible-playbook playbooks/install_k3s.yml

4. 部署ELK
ansible-playbook playbooks/deploy_elk.yml
