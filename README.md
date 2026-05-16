# Python (Flask) Web App with MySQL and Key Vault

Artists Booking Venues powered by Python (Flask) and MySQL Database.
There is no user authentication or per-user data stored.

![Screenshot of website landing page](./repo-thumbnail.png)

The project is designed for deployment on Azure App Service with a MySQL flexible server. See deployment instructions below.

[![Open in GitHub Codespaces](https://github.com/codespaces/badge.svg)](https://codespaces.new/john0isaac/flask-webapp-mysql-db?devcontainer_path=.devcontainer/devcontainer.json)

![Architecture Diagram: App Service, MySQL server, Key Vault](./architecture-diagram.png)

## Local Development

1. **Download the project starter code locally**

    ```bash
    git clone https://github.com/john0isaac/flask-webapp-mysql-db.git
    cd flask-webapp-mysql-db
    ```

2. **Install, initialize and activate a virtualenv using:**

    ```bash
    pip install virtualenv
    python -m virtualenv venv
    source venv/bin/activate
    ```

    >**Note** - In Windows, the `venv` does not have a `bin` directory. Therefore, you'd use the analogous command shown below:

    ```bash
    source venv\Scripts\activate
    ```

3. **Install the dependencies:**

    ```bash
    pip install -r requirements.txt
    ```

4. **Run the development server:**

    ```bash
    export FLASK_APP=app.py
    export FLASK_ENV=development
    export FLASK_DEBUG=true
    flask run --reload
    ```

    **For Windows, use [`setx`](https://learn.microsoft.com/windows-server/administration/windows-commands/setx) command shown below:**

   ```powershell
    setx FLASK_APP app.py
    setx FLASK_ENV development
    setx FLASK_DEBUG true
    flask run --reload
    ```
   
6. **Verify on the Browser**

Navigate to project homepage [http://127.0.0.1:5000/](http://127.0.0.1:5000/) or [http://localhost:5000](http://localhost:5000)

### Connect to your Database locally

This application is configured to work without connecting a database for the pages that doesn't interact with it. If you want to enable database operations you'll need to connect to a database.

- **Run the following commands to connect to a database:**

    ```bash
    export DEPLOYMENT_LOCATION=local
    export DB_USER=
    export DB_PASSWORD=
    export DB_HOST=
    export DB_NAME=
    ```

- **For windows, add configuration using:**

    ```powershell
    setx DEPLOYMENT_LOCATION local
    setx DB_USER user
    setx DB_PASSWORD password
    setx DB_HOST host
    setx DB_NAME dbname
    ```

- **Run the development server again.**

## Deployment

Repo này có **2 cách deploy** lên Azure: bằng Azure Portal (giao diện web) hoặc dùng Azure Developer CLI (terminal). Bạn có thể chọn cách phù hợp nhất với mình.

---

### ✅ Cách 1: Dùng Azure Portal (không cần terminal)

**Bước 1 — Fork repo về GitHub của bạn**
Vào **github.com/john0isaac/flask-webapp-mysql-db** → nhấn nút **Fork** (góc trên phải) → **Create fork**

**Bước 2 — Tạo MySQL Database trên Azure Portal**
1. Vào **portal.azure.com**
2. Tìm **"Azure Database for MySQL"** → **Create** → **Flexible Server**
3. Điền:
   - Server name: `flask-mysql-server`
   - Region: `Southeast Asia` (Hoặc khu vực gần bạn nhất)
   - MySQL version: `8.0`
   - Admin username / password: tự đặt, **nhớ lưu lại**
4. Tab **Networking** → bật **Allow public access** → **Add current client IP** (để có thể import data từ máy tính của bạn).
5. **Review + Create** → **Create**

Sau khi tạo xong, vào server → **Databases** → **Add** → đặt tên `flask_db`

**Bước 3 — Tạo App Service**
1. Tìm **"App Services"** → **Create**
2. Điền:
   - Name: `flask-artist-app` *(thành domain của bạn)*
   - Runtime stack: **Python 3.11** (hoặc bản Python tương ứng)
   - OS: **Linux**
   - Region: **Southeast Asia**
   - Plan: **F1 Free** (miễn phí để học)
3. **Review + Create** → **Create**

**Bước 4 — Kết nối GitHub và deploy tự động**
1. Vào App Service vừa tạo → menu trái **Deployment Center**
2. Source: chọn **GitHub**
3. Đăng nhập GitHub → chọn repo bạn vừa fork
4. Branch: **main**
5. Nhấn **Save** → Azure tự động build và deploy! ✅

**Bước 5 — Cấu hình biến môi trường (thông tin DB)**
Vào App Service → **Configuration** → **Application settings** → **New application setting**, thêm lần lượt:

| Name | Value |
|---|---|
| `DEPLOYMENT_LOCATION` | `azure` |
| `AZURE_MYSQL_HOST` | `flask-mysql-server.mysql.database.azure.com` |
| `AZURE_MYSQL_USER` | username đã tạo ở bước 2 |
| `AZURE_MYSQL_PASSWORD` | password đã tạo ở bước 2 |
| `AZURE_MYSQL_NAME` | `flask_db` |

Nhấn **Save** → App tự động restart.

*(Lưu ý: Tên biến môi trường phải chính xác như trên vì code trong file `environment/azure_production.py` đọc các biến này).*

**Bước 6 — Import database**
1. Mở file **`mock_up_data.sql`** trong repo. Nếu chưa sửa, hãy đổi dòng `GRANT ALL PRIVILEGES ON DATABASE fyyur TO john;` thành `GRANT ALL PRIVILEGES ON flask_db.* TO 'username_cua_ban';` (hoặc có thể xóa dòng đó đi nếu user admin đã có full quyền).
2. Dùng terminal trên máy tính hoặc Azure Cloud Shell để import dữ liệu:

```bash
mysql -h flask-mysql-server.mysql.database.azure.com \
      -u username_cua_ban -p flask_db < mock_up_data.sql
```

**Kết quả:** Truy cập `https://flask-artist-app.azurewebsites.net` là xong! 🎉

---

### ✅ Cách 2: Dùng Azure Developer CLI (nhanh hơn, chỉ 3 lệnh)

Nếu bạn muốn dùng terminal, đây là cách **siêu nhanh** đã được cấu hình sẵn trong thư mục `infra/`.

1. **Cài Azure Developer CLI trước**
   - Windows: `winget install microsoft.azd`
   - Mac: `brew install azd`

2. **Khởi tạo project**
   ```bash
   azd init -t john0isaac/flask-webapp-mysql-db
   ```

3. **Deploy lên Azure**
   ```bash
   azd up
   ```
   Lệnh này tự động tạo App Service, MySQL, Key Vault và deploy code luôn!

---

### So sánh 2 cách

| | Portal (Cách 1) | Azure CLI (Cách 2) |
|---|---|---|
| Cần terminal | ❌ Không | ✅ Có |
| Số bước | Nhiều hơn | 3 lệnh |
| Học được nhiều hơn | ✅ Hiểu từng phần | ❌ Tự động hết |
| Phù hợp | Người mới học | Muốn nhanh |

## Security

It is important to secure the databases in web applications to prevent unwanted data access.
This infrastructure uses the following mechanisms to secure the MySQL database:

* Azure Firewall: The database is accessible only from other Azure IPs, not from public IPs. (Note that includes other customers using Azure).
* Admin Username: Randomly generated and stored in Key Vault (nếu dùng Cách 2).
* Admin Password: Randomly generated and stored in Key Vault (nếu dùng Cách 2).
* MySQL Version: Latest available on Azure, version 8.0, which includes security improvements.

⚠️ For even more security, consider using an Azure Virtual Network to connect the Web App to the Database.

## Costs

Pricing varies per region and usage, so it isn't possible to predict exact costs for your usage.

You can try the [Azure pricing calculator](https://azure.microsoft.com/pricing/calculator/) for the resources:

* Azure App Service: Free Tier with shared CPU cores, 1 GB RAM. [Pricing](https://azure.microsoft.com/pricing/details/app-service/linux/)
* MySQL Flexible Server: Burstable Tier with 1 CPU core, 20GB storage. Pricing is hourly. [Pricing](https://azure.microsoft.com/pricing/details/mysql/)
* Key Vault: Standard tier with 2 secrets. Vaults are offered in two service tiers—standard and premium. [Pricing](https://azure.microsoft.com/pricing/details/key-vault/)

⚠️ To avoid unnecessary costs, remember to take down your app if it's no longer in use, either by deleting the resource group in the Portal or running `azd down`.
