# Reflection

## 1. What are the key differences between unary, server streaming, and bi-directional streaming RPC (Remote Procedure Call) methods, and in what scenarios would each be most suitable?

**Unary:** Unary merupakan bentuk RPC yang paling dasar, di mana client mengirim satu permintaan dan server hanya mengembalikan satu balasan. Pola ini cocok untuk proses yang sederhana dan cepat, seperti request pembayaran pada `PaymentService`.

**Server Streaming:** Server streaming digunakan ketika satu request dari client dapat menghasilkan beberapa response dari server secara berurutan. Biasanya pola ini cocok untuk menampilkan data yang panjang atau bertahap, misalnya daftar riwayat transaksi pada `TransactionService`.

**Bi-directional Streaming:** Bi-directional streaming memungkinkan client dan server berkomunikasi dua arah secara terus-menerus dalam satu koneksi. Pola ini sesuai untuk fitur yang membutuhkan interaksi real-time, seperti pengiriman pesan pada `ChatService`.
aksi real-time, seperti pengiriman pesan pada ChatService.

## 2.  What are the potential security considerations involved in implementing a gRPC service in Rust, particularly regarding authentication, authorization, and data encryption?

Dalam implementasi gRPC di Rust, authentication diperlukan untuk memastikan client yang mengakses service benar-benar valid, misalnya menggunakan JWT, API key, atau metadata pada request. Authorization juga penting agar setiap user hanya bisa mengakses data atau aksi yang sesuai dengan hak aksesnya. Selain itu, data encryption perlu diterapkan menggunakan TLS agar komunikasi antara client dan server tidak mudah disadap. Untuk service seperti payment, validasi input, logging, dan audit trail juga penting karena data transaksi bersifat sensitif.

## 3. What are the potential challenges or issues that may arise when handling bidirectional streaming in Rust gRPC, especially in scenarios like chat applications?

Bidirectional streaming memiliki tantangan karena client dan server dapat mengirim data secara bersamaan. Pada aplikasi chat, server harus mampu mengelola banyak koneksi aktif, menyimpan state pengguna, dan meneruskan pesan ke penerima yang tepat.

Masalah lain yang bisa muncul adalah concurrency, race condition, client disconnect, dan backpressure. Jika pesan dikirim terlalu cepat sementara penerima lambat memproses, buffer bisa penuh dan menyebabkan performa aplikasi menurun.

## 4. What are the advantages and disadvantages of using the tokio_stream::wrappers::ReceiverStream for streaming responses in Rust gRPC services?

`tokio_stream::wrappers::ReceiverStream` memudahkan developer mengubah channel Tokio menjadi stream yang bisa dikirim sebagai response gRPC. Ini membuat implementasi server streaming lebih praktis, terutama ketika data dikirim dari task asynchronous lain.

Kekurangannya, penggunaan channel tetap membutuhkan pengelolaan buffer dan error handling yang baik. Jika buffer terlalu kecil, pengiriman bisa terhambat, sedangkan jika terlalu besar, penggunaan memori bisa menjadi tidak efisien.

## 5. In what ways could the Rust gRPC code be structured to facilitate code reuse and modularity, promoting maintainability and extensibility over time?

Kode Rust gRPC dapat dibuat lebih modular dengan memisahkan generated proto, service handler, business logic, dan konfigurasi server. Dengan begitu, file utama tidak terlalu penuh dan setiap bagian memiliki tanggung jawab yang jelas. Misalnya, `MyPaymentService` cukup menangani request dan response gRPC, sedangkan logika pembayaran dipindahkan ke module lain seperti `payment_processor`. Struktur seperti ini membuat kode lebih mudah diuji, digunakan ulang, dan dikembangkan.

## 6. In the MyPaymentService implementation, what additional steps might be necessary to handle more complex payment processing logic?

Implementasi `MyPaymentService` saat ini masih sangat sederhana karena hanya menerima request dan langsung mengembalikan `success: true`. Untuk kasus nyata, service perlu melakukan validasi request, mengecek data pembayaran, dan memastikan user memiliki izin untuk melakukan transaksi. Selain itu, service juga perlu terhubung ke database dan payment gateway, menangani error, retry, timeout, serta menyimpan status transaksi. Idempotency juga penting agar pembayaran tidak diproses dua kali ketika request dikirim ulang.

## 7. What impact does the adoption of gRPC as a communication protocol have on the overall architecture and design of distributed systems, particularly in terms of interoperability with other technologies and platforms?

Penggunaan gRPC membuat arsitektur sistem lebih contract-driven karena komunikasi antar-service didefinisikan melalui file `.proto`. Hal ini membantu menjaga konsistensi format data antara client dan server.

gRPC juga mendukung banyak bahasa pemrograman, sehingga service yang dibuat dengan Rust dapat berkomunikasi dengan service lain yang dibuat menggunakan Go, Java, Python, dan sebagainya. Namun, kita tetap perlu memperhatikan versioning proto agar perubahan API tidak merusak client lama.

## 8. What are the advantages and disadvantages of using HTTP/2, the underlying protocol for gRPC, compared to HTTP/1.1 or HTTP/1.1 with WebSocket for REST APIs?

HTTP/2 memiliki kelebihan seperti multiplexing, header compression, dan koneksi yang lebih efisien. Hal ini membuat gRPC cocok untuk komunikasi antar-service yang membutuhkan performa tinggi dan mendukung streaming.

Namun, gRPC lebih sulit di-debug dibanding REST biasa karena menggunakan format biner Protocol Buffers, bukan JSON yang mudah dibaca. WebSocket lebih fleksibel untuk komunikasi real-time di browser, sedangkan gRPC lebih cocok untuk komunikasi service-to-service.

## 9. How does the request-response model of REST APIs contrast with the bidirectional streaming capabilities of gRPC in terms of real-time communication and responsiveness?

REST API umumnya menggunakan model request-response, yaitu client mengirim request lalu server mengirim satu response. Untuk kebutuhan real-time, REST biasanya memerlukan polling, long polling, SSE, atau WebSocket.

Sebaliknya, bidirectional streaming pada gRPC memungkinkan koneksi tetap terbuka sehingga client dan server bisa saling mengirim data kapan saja. Ini membuat gRPC lebih responsif untuk aplikasi seperti chat, notifikasi, dan live monitoring.

## 10. What are the implications of the schema-based approach of gRPC, using Protocol Buffers, compared to the more flexible, schema-less nature of JSON in REST API payloads?

gRPC menggunakan Protocol Buffers yang berbasis schema, sehingga struktur data harus didefinisikan terlebih dahulu di file `.proto`. Keuntungannya adalah komunikasi menjadi lebih konsisten, type-safe, dan efisien.

Sebaliknya, JSON pada REST lebih fleksibel dan mudah dibaca manusia, sehingga lebih mudah digunakan untuk debugging atau API publik. Namun, karena tidak memiliki schema yang ketat, kesalahan tipe data atau field yang hilang lebih sering baru terlihat saat runtime.
