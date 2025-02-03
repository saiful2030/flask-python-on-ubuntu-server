# Dillinger
## Flask di Ubuntu Server

## Installation

#### 1. Install Dependensi
Jalankan perintah berikut untuk memastikan server memiliki semua dependensi yang diperlukan:
```sh
sudo apt update && sudo apt upgrade -y
```
```sh
sudo apt install python3 python3-pip python3-venv nginx -y
```
#### 2. Clone atau Buat Aplikasi Flask
Masuk ke direktori tempat Anda ingin menyimpan aplikasi:
```sh
cd /var/www/
```
```sh
sudo mkdir flaskapp
```
```sh
sudo chown $USER:$USER flaskapp
```
```sh
cd flaskapp
```
Buat virtual environment:
```sh
python3 -m venv venv
```
```sh
source venv/bin/activate
```
Install Flask:
```sh
pip install flask gunicorn
````
Buat file aplikasi contoh (app.py):
```sh
from flask import Flask

app = Flask(__name__)

@app.route("/")
def home():
    return "Hello, Flask on Ubuntu!"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)

````
#### 3. Menjalankan Flask dengan Gunicorn
Jalankan Gunicorn untuk memastikan aplikasi berjalan:
```sh
gunicorn --bind 0.0.0.0:5000 app:app
````
Cek apakah aplikasi berjalan dengan:
```sh
curl http://127.0.0.1:5000
````
Tekan CTRL+C untuk menghentikan.
#### 4. Konfigurasi Gunicorn dengan Systemd
Buat file service untuk Gunicorn:
```sh
sudo nano /etc/systemd/system/flaskapp.service
````
Tambahkan isi berikut:
```sh
[Unit]
Description=Gunicorn instance to serve Flask App
After=network.target

[Service]
User=www-data
Group=www-data
WorkingDirectory=/var/www/flaskapp
ExecStart=/var/www/flaskapp/venv/bin/gunicorn --workers 3 --bind unix:/var/www/flaskapp/flaskapp.sock -m 007 app:app

[Install]
WantedBy=multi-user.target

````
Simpan dan keluar (CTRL+X, Y, Enter).
Reload systemd dan jalankan service:
```sh
sudo systemctl daemon-reload
````
```sh
sudo systemctl start flaskapp
````
```sh
sudo systemctl enable flaskapp
````
Cek statusnya:
```sh
sudo systemctl status flaskapp
````
#### 5. Konfigurasi Nginx sebagai Reverse Proxy
Buat file konfigurasi Nginx:
```sh
sudo nano /etc/nginx/sites-available/flaskapp
````
Tambahkan isi berikut:
```sh
server {
    listen 80;
    server_name your_domain_or_ip;

    location / {
        include proxy_params;
        proxy_pass http://unix:/var/www/flaskapp/flaskapp.sock;
    }
}

````
Simpan dan keluar.
Aktifkan konfigurasi:
```sh
sudo ln -s /etc/nginx/sites-available/flaskapp /etc/nginx/sites-enabled
````
```sh
sudo nginx -t
````
Jika tidak ada error, restart Nginx:
```sh
sudo systemctl restart nginx
````
#### 6. Uji Aplikasi
Buka browser dan akses:
```sh
http://your_domain_or_ip
````
Jika semuanya berjalan lancar, aplikasi Flask Anda sudah berhasil dihosting di Ubuntu Server!
Jika ada kendala atau ingin tambahan seperti [HTTPS dengan Let's Encrypt](https://github.com/saiful2030/Hosting-Flask-di-Ubuntu-Server-dengan-Let-s-Encrypt)
