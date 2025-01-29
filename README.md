# openproject
docker-compose file for openproject installation
นี่คือตัวอย่างไฟล์ docker-compose.yml ที่แก้ไขแล้ว สำหรับ OpenProject พร้อมคำอธิบายการตั้งค่าแบบละเอียด:

version: '3'

services:
  postgres:
    image: postgres:16
    container_name: openproject-postgres
    restart: always
    environment:
      POSTGRES_USER: openproject
      POSTGRES_PASSWORD: your_secure_password_here  # เปลี่ยนรหัสผ่านนี้
      POSTGRES_DB: openproject
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - openproject-network

  memcached:
    image: memcached:latest
    container_name: openproject-memcached
    restart: always
    networks:
      - openproject-network

  openproject:
    image: openproject/openproject:15
    container_name: openproject
    restart: always
    depends_on:
      - postgres
      - memcached
    ports:
      - "8686:80"  # แม็พพอร์ต 80 ใน container ไปยังพอร์ต 8686 ของ host
    environment:
      OPENPROJECT_HOST__NAME: "192.168.1.6"  # เปลี่ยนเป็นโดเมนของคุณถ้ามี
      OPENPROJECT_HTTPS: "false"
      DATABASE_URL: "postgres://openproject:your_secure_password_here@postgres:5432/openproject"  # ต้องตรงกับ postgres service
      OPENPROJECT_SECRET_KEY_BASE: "generate_a_secure_random_key_here"  # สร้างด้วยคำสั่ง: openssl rand -hex 64
      OPENPROJECT_CACHE__STORE: "memcache"
      OPENPROJECT_CACHE__MEMCACHE__SERVER: "memcached:11211"
    volumes:
      - openproject-data:/var/openproject/assets
      - openproject-static:/var/openproject/public
    networks:
      - openproject-network

volumes:
  postgres-data:
    driver: local
  openproject-data:
    driver: local
  openproject-static:
    driver: local

networks:
  openproject-network:
    driver: bridge

----------------------------------------------------------------------------------------------    
🔥 วิธีใช้งาน:
แก้ไขค่าต่างๆ ในไฟล์:

POSTGRES_PASSWORD: ตั้งรหัสผ่านสำหรับ PostgreSQL

DATABASE_URL: ใส่รหัสผ่านให้ตรงกับ POSTGRES_PASSWORD

OPENPROJECT_SECRET_KEY_BASE: สร้างด้วยคำสั่ง
---------------------------------------------------------------------------------------------- 
bash:

openssl rand -hex 64
เริ่มระบบด้วยคำสั่ง:
---------------------------------------------------------------------------------------------- 
bash

docker-compose up -d
ตรวจสอบ logs เพื่อดูสถานะ:
---------------------------------------------------------------------------------------------- 
bash

docker-compose logs -f openproject
---------------------------------------------------------------------------------------------- 
📌 หมายเหตุสำคัญ:
รอให้ระบบเริ่มทำงานสมบูรณ์:
OpenProject อาจใช้เวลา 2-5 นาทีในการเริ่มทำงานครั้งแรก (ขึ้นอยู่กับสเปคเซิร์ฟเวอร์)

ตรวจสอบการเชื่อมต่อ PostgreSQL:
หากพบข้อผิดพลาดใน logs เกี่ยวกับ database:

bash

docker exec -it openproject-postgres psql -U openproject -d openproject
ไฟร์วอลล์:
ตรวจสอบว่าเปิดพอร์ต 8686 แล้ว:
---------------------------------------------------------------------------------------------- 
bash

ufw allow 8686/tcp
ทดสอบจากเครื่อง client:
---------------------------------------------------------------------------------------------- 
bash
Copy
curl -v http://192.168.1.6:8686
# หรือเปิดเบราว์เซอร์ที่ http://192.168.1.6:8686
