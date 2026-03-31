# BambangShop Receiver App
Tutorial and Example for Advanced Programming 2024 - Faculty of Computer Science, Universitas Indonesia

---

## About this Project
In this repository, we have provided you a REST (REpresentational State Transfer) API project using Rocket web framework.

This project consists of four modules:
1.  `controller`: this module contains handler functions used to receive request and send responses.
    In Model-View-Controller (MVC) pattern, this is the Controller part.
2.  `model`: this module contains structs that serve as data containers.
    In MVC pattern, this is the Model part.
3.  `service`: this module contains structs with business logic methods.
    In MVC pattern, this is also the Model part.
4.  `repository`: this module contains structs that serve as databases.
    You can use methods of the struct to get list of objects, or operating an object (create, read, update, delete).

This repository provides a Rocket web framework skeleton that you can work with.

As this is an Observer Design Pattern tutorial repository, you need to implement a feature: `Notification`.
This feature will receive notifications of creation, promotion, and deletion of a product, when this receiver instance is subscribed to a certain product type.
The notification will be sent using HTTP POST request, so you need to make the receiver endpoint in this project.

## API Documentations

You can download the Postman Collection JSON here: https://ristek.link/AdvProgWeek7Postman

After you download the Postman Collection, you can try the endpoints inside "BambangShop Receiver" folder.

Postman is an installable client that you can use to test web endpoints using HTTP request.
You can also make automated functional testing scripts for REST API projects using this client.
You can install Postman via this website: https://www.postman.com/downloads/

## How to Run in Development Environment
1.  Set up environment variables first by creating `.env` file.
    Here is the example of `.env` file:
    ```bash
    ROCKET_PORT=8001
    APP_INSTANCE_ROOT_URL=http://localhost:${ROCKET_PORT}
    APP_PUBLISHER_ROOT_URL=http://localhost:8000
    APP_INSTANCE_NAME=Safira Sudrajat
    ```
    Here are the details of each environment variable:
    | variable                | type   | description                                                     |
    |-------------------------|--------|-----------------------------------------------------------------|
    | ROCKET_PORT             | string | Port number that will be listened by this receiver instance.    |
    | APP_INSTANCE_ROOT_URL   | string | URL address where this receiver instance can be accessed.       |
    | APP_PUUBLISHER_ROOT_URL | string | URL address where the publisher instance can be accessed.       |
    | APP_INSTANCE_NAME       | string | Name of this receiver instance, will be shown on notifications. |
2.  Use `cargo run` to run this app.
    (You might want to use `cargo check` if you only need to verify your work without running the app.)
3.  To simulate multiple instances of BambangShop Receiver (as the tutorial mandates you to do so),
    you can open new terminal, then edit `ROCKET_PORT` in `.env` file, then execute another `cargo run`.

    For example, if you want to run 3 (three) instances of BambangShop Receiver at port `8001`, `8002`, and `8003`, you can do these steps:
    -   Edit `ROCKET_PORT` in `.env` to `8001`, then execute `cargo run`.
    -   Open new terminal, edit `ROCKET_PORT` in `.env` to `8002`, then execute `cargo run`.
    -   Open another new terminal, edit `ROCKET_PORT` in `.env` to `8003`, then execute `cargo run`.

## Mandatory Checklists (Subscriber)
-   [ ] Clone https://gitlab.com/ichlaffterlalu/bambangshop-receiver to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [ ] Commit: `Create Notification model struct.`
    -   [ ] Commit: `Create SubscriberRequest model struct.`
    -   [ ] Commit: `Create Notification database and Notification repository struct skeleton.`
    -   [ ] Commit: `Implement add function in Notification repository.`
    -   [ ] Commit: `Implement list_all_as_string function in Notification repository.`
    -   [ ] Write answers of your learning module's "Reflection Subscriber-1" questions in this README.
-   **STAGE 3: Implement services and controllers**
    -   [ ] Commit: `Create Notification service struct skeleton.`
    -   [ ] Commit: `Implement subscribe function in Notification service.`
    -   [ ] Commit: `Implement subscribe function in Notification controller.`
    -   [ ] Commit: `Implement unsubscribe function in Notification service.`
    -   [ ] Commit: `Implement unsubscribe function in Notification controller.`
    -   [ ] Commit: `Implement receive_notification function in Notification service.`
    -   [ ] Commit: `Implement receive function in Notification controller.`
    -   [ ] Commit: `Implement list_messages function in Notification service.`
    -   [ ] Commit: `Implement list function in Notification controller.`
    -   [ ] Write answers of your learning module's "Reflection Subscriber-2" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Subscriber) Reflections

#### Reflection Subscriber-1

1. RwLock<> diperlukan karena Vec of Notifications diakses dari multiple threads secara bersamaan, sehingga tanpa lock bisa terjadi data race. Alasan kita tidak pakai Mutex<> adalah karena Mutex<> hanya mengizinkan satu thread mengakses data dalam satu waktu (baik untuk read maupun write). Sedangkan RwLock<> lebih fleksibel: banyak thread boleh read secara bersamaan (selama tidak ada thread yang sedang write), tetapi jika ada yang mau write, harus menunggu semua yang read selesai dahulu. Karena operasi read list notifikasi jauh lebih sering terjadi dibanding operasi write, RwLock<> lebih efisien di case ini.

2. Di Java, kita bisa bebas mengubah static variable melalui static function tanpa masalah karena Java mengurus keamanannya saat runtime. Di Rust, semua pengecekan memory safety dan thread safety dilakukan saat compile time, bukan runtime. Rust tidak mengizinkan mutasi static variable secara langsung karena variable tersebut hidup sepanjang program berjalan dan bisa diakses dari multiple threads sekaligus, yang berpotensi menyebabkan data race. Oleh karena itu, kita butuh lazy_static untuk inisialisasi yang aman, dan membungkusnya dengan RwLock<> atau DashMap yang menjaga dan mengatur akses agar tetap thread-safe.

#### Reflection Subscriber-2

1. Iya, saya membuka src/lib.rs dan beberapa file lainnya untuk memahami struktur project secara keseluruhan. Di src/lib.rs saya melihat bagaimana semua module (model, repository, service, controller) di-register dan dihubungkan satu sama lain agar bisa dipakai di seluruh project. Ini menarik karena di Java biasanya tidak ada file seperti ini, setiap class langsung bisa diimport dari packagenya. Di Rust, kita perlu secara eksplisit mendeklarasikan module mana saja yang ada dan bagaimana hierarkinya, dan lib.rs adalah seperti entrance dari semua itu.

2. Dengan Observer pattern, menambahkan Subscriber baru jadi sangat mudah, cukup jalankan instance Receiver baru di port berbeda lalu subscribe ke product type yang diinginkan, tanpa perlu mengubah kode Main app sama sekali. Tapi kalau kita spawn lebih dari satu instance Main app, situasinya jadi lebih rumit. Setiap instance punya SUBSCRIBERS DashMapnya sendiri di memory masing-masing, sehingga kalau Subscriber mendaftar ke instance pertama, instance kedua tidak akan tau dan tidak akan mengirim notifikasi ke Subscriber tersebut. Untuk mengatasi ini dibutuhkan shared storage eksternal seperti database yang bisa diakses semua instance, yang berarti perubahan arsitektur yang cukup signifikan.

3. Saya mencoba fitur Tests di Postman, yaitu fitur yang memungkinkan kita menulis script JavaScript sederhana untuk memvalidasi response secara otomatis setelah request dikirim, misalnya mengecek apakah status code-nya 200, atau apakah response body mengandung field tertentu. Fitur ini menurut saya sangat berguna karena kita tidak perlu mengecek response secara manual setiap kali melakukan test. Untuk Group Project, ini akan sangat membantu terutama saat regression testing. Kita bisa jalankan seluruh collection sekaligus dan langsung tau apakah ada endpoint yang bermasalah setelah ada perubahan kode.
