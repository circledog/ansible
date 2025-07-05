# 使用 Ansible 部署 ELK Stack

這是一個使用 Ansible 自動在 Ubuntu 22.04/24.04 伺服器上部署完整 ELK (Elasticsearch, Logstash, Kibana) Stack 的專案。

## 先決條件

1.  **Ansible 控制節點**: 你需要在你的本機或一台管理伺服器上安裝 Ansible。
2.  **目標伺服器**: 一台或多台 Ubuntu 22.04/24.04 伺服器，並且你的控制節點可以透過 SSH 連線。
3.  **SSH 金鑰**: 建議設定從控制節點到目標伺服器的無密碼 SSH 連線，以方便 Ansible 執行。

## 如何使用

1.  **編輯 `inventory` 檔案**:
    打開 `inventory` 檔案，將 `your_server_ip` 替換為你目標 Ubuntu 伺服器的實際 IP 位址。將 `your_username` 替換為你用來 SSH 登入的用戶名。
    例如：
    ```ini
    [elk_servers]
    192.168.56.10 ansible_user=ubuntu
    ```

2.  **執行 Playbook**:
    在專案的根目錄下，執行以下命令來開始部署：
    ```bash
    ansible-playbook -i inventory elk-playbook.yml
    ```

3.  **驗證安裝**:
    *   **Elasticsearch**: 在你的終端機執行 `curl http://your_server_ip:9200`。你應該會看到一個包含版本資訊的 JSON 回應。
    *   **Kibana**: 在你的瀏覽器中開啟 `http://your_server_ip:5601`。你應該會看到 Kibana 的載入頁面。
    *   **Logstash**: Logstash 會在背景執行，監聽來自 Beats 的日誌 (預設設定在 5044 連接埠)。

## 部署詳情

這個 Playbook 會執行以下任務：

1.  **Common Role**:
    *   更新 APT 快取。
    *   安裝 ELK 需要的共用套件，例如 `apt-transport-https` 和 `openjdk-17-jdk`。

2.  **Elasticsearch Role**:
    *   匯入 Elastic 的 GPG 金鑰。
    *   新增 Elastic 的 APT 套件庫。
    *   安裝並設定 Elasticsearch。
    *   設定為單節點模式 (`discovery.type: single-node`)。
    *   監聽所有網路介面 (`network.host: 0.0.0.0`)，以便 Kibana 和 Logstash 連線。
    *   啟動並設定開機自啟 Elasticsearch 服務。

3.  **Kibana Role**:
    *   安裝並設定 Kibana。
    *   設定 Kibana 連線到本機的 Elasticsearch 實例。
    *   監聽所有網路介面 (`server.host: "0.0.0.0"`)，以便你可以從任何地方存取。
    *   啟動並設定開機自啟 Kibana 服務。

4.  **Logstash Role**:
    *   安裝並設定 Logstash。
    *   建立一個基本的設定檔 (`/etc/logstash/conf.d/02-beats-input.conf`)，它會：
        *   從 5044 連接埠接收 Beats 的輸入 (例如 Filebeat)。
        *   將接收到的日誌輸出到 Elasticsearch。
    *   啟動並設定開機自啟 Logstash 服務。
