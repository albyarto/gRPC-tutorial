# gRPC Tutorial (Modul 08)
Tutorial and Example for Advanced Programming 2024 - Faculty of Computer Science, Universitas Indonesia

Muhammad Albyarto Ghazali (2306241695)

---

## Reflection

>What are the key differences between unary, server streaming, and bi-directional streaming RPC (Remote Procedure Call) methods, and in what scenarios would each be most suitable?

Berdasarkan modul yang saya baca, dalam gRPC terdapat beberapa pola komunikasi yang berbeda

* **Unary RPC**: Klien mengirim satu permintaan dan menerima satu respons dari server. Cocok untuk operasi sederhana seperti autentikasi (login) di mana klien mengirim kredensial dan server mengembalikan token akses.

* **Server Streaming RPC**: Klien mengirim satu permintaan dan menerima aliran data dari server. Berguna untuk mengirimkan data besar atau pembaruan real-time, seperti riwayat transaksi atau notifikasi.

* **Bidirectional Streaming RPC**: Klien dan server saling mengirim aliran data secara bersamaan. Ideal untuk aplikasi interaktif seperti chat real-time, di mana klien dan server dapat saling bertukar pesan secara terus-menerus selama koneksi terbuka.

>What are the potential security considerations involved in implementing a gRPC service in Rust, particularly regarding authentication, authorization, and data encryption?

Beberapa aspek penting keamanan dalam gRPC:

* **Authentication**: Memastikan bahwa hanya klien yang sah dapat mengakses layanan.

* **Authorization**: Menentukan hak akses klien terhadap sumber daya tertentu.

* **Data Encryption**: Mengamankan data yang dikirim melalui jaringan menggunakan TLS untuk mencegah penyadapan atau manipulasi data.

gRPC memiliki protokol TLS (Transport Layer Security) untuk mengamankan komunikasi antara client dan server. Di Rust, biasanya TLS dapat diaktifkan dengan mengkonfigurasi server untuk memuat sertifikat digital dan private key melalui pustaka seperti `tonic` dan `rustls`. Dengan menggunakan TLS, data yang dikirimkan akan dienkripsi dan dijamin kerahasiaannya selama transmisi. Selain itu, pengembang dapat menambahkan lapisan keamanan tambahan seperti autentikasi token JWT atau API key untuk memastikan bahwa hanya klien yang sah yang bisa mengakses layanan tertentu. Keamanan juga bisa ditingkatkan dengan penggunaan firewall, pembatasan IP, dan validasi input untuk mencegah serangan seperti injection atau DoS.

>What are the potential challenges or issues that may arise when handling bidirectional streaming in Rust gRPC, especially in scenarios like chat applications?

Tantangan utama dalam menangani bidirectional streaming di Rust gRPC adalah mengelola komunikasi dua arah secara asynchronous tanpa blocking atau race condition. Dalam aplikasi seperti chat, diperlukan sistem yang dapat menerima dan mengirim pesan secara simultan dan real-time, yang menuntut penggunaan asynchronous primitives seperti `tokio::mpsc` dan sink/stream combinators. Selain itu, ada tantangan dalam mengatur buffer, menghindari memory leak, serta menjaga sinkronisasi state antara client dan server. Penggunaan tokio runtime dan task spawning perlu dikelola dengan hati-hati agar tidak menimbulkan overhead atau kesalahan komunikasi yang sulit dilacak.

>What are the advantages and disadvantages of using the tokio_stream::wrappers::ReceiverStream for streaming responses in Rust gRPC services?

`ReceiverStream` adalah wrapper untuk channel `tokio::sync::mpsc`, yang memungkinkan server mengirim stream data kepada klien dengan cara yang mudah dalam ekosistem asynchronous Rust. Keunggulannya adalah kemudahan integrasi dengan sistem producer-consumer, pemisahan logic bisnis dari logic streaming, serta compatibility langsung dengan interface stream pada `tonic`. Namun, kekurangannya adalah dependency pada channel yang harus dikelola dengan hati-hati agar tidak mengalami deadlock atau kehabisan kapasitas. Selain itu, error handling yang kompleks dan kebutuhan synchronization antar task bisa menjadi kendala saat sistem menjadi lebih besar atau parallel.
  
>In what ways could the Rust gRPC code be structured to facilitate code reuse and modularity, promoting maintainability and extensibility over time?

Untuk mencapai modularitas dan kemudahan perawatan dalam proyek Rust gRPC, struktur kode sebaiknya dipisah berdasarkan tanggung jawab, seperti memisahkan definisi protobuf (modul `proto`), implementasi layanan (`services` atau `handlers`), model data internal (`models`), serta utilitas umum (`utils`). Selain itu, penggunaan trait dan implementasi yang terpisah untuk setiap service membantu dalam menguji, mengganti, atau mengembangkan modul tanpa mengganggu bagian lainnya. Penggunaan `mod.rs` dan pengorganisasian crate dengan baik juga mendukung scalability proyek, terutama saat proyek berkembang menjadi microservices.

>In the MyPaymentService implementation, what additional steps might be necessary to handle more complex payment processing logic?

Dalam implementasi `MyPaymentService`, untuk menangani logic pembayaran yang lebih kompleks, perlu ditambahkan validasi data transaksi/validasi input, integrasi dengan gateway pembayaran pihak ketiga, serta sistem manajemen status transaksi. Harus ada mekanisme untuk menangani retry, rollback, dan log pencatatan transaksi untuk kebutuhan audit. Selain itu, penting untuk menyertakan sistem authentication dan authorization yang ketat, serta memastikan transaksi diproses secara atomic untuk mencegah duplikasi atau kehilangan dana. Validasi terhadap fraud dan integrasi dengan sistem notifikasi juga penting meningkatkan service.

>What impact does the adoption of gRPC as a communication protocol have on the overall architecture and design of distributed systems, particularly in terms of interoperability with other technologies and platforms?

Penggunaan gRPC sebagai communication protocol membawa dampak besar dalam distributed systems. Dengan menggunakan Protocol Buffers dan HTTP/2, gRPC menawarkan komunikasi lintas platform yang efisien dan terstandarisasi. Sistem dapat dikembangkan dalam berbagai bahasa pemrograman tanpa kehilangan kompatibilitas. Namun, interoperabilitas dengan sistem yang hanya mendukung REST atau protokol konvensional bisa menjadi tantangan, yang seringkali mengharuskan penggunaan gateway seperti Envoy atau grpc-gateway. Secara keseluruhan, gRPC mendorong pendekatan berbasis kontrak yang kuat, meningkatkan integritas komunikasi antar layanan, tetapi membutuhkan tooling dan perencanaan lebih matang dalam integrasi dengan sistem.

>What are the advantages and disadvantages of using HTTP/2, the underlying protocol for gRPC, compared to HTTP/1.1 or HTTP/1.1 with WebSocket for REST APIs?

HTTP/2, yang menjadi dasar gRPC, memiliki keunggulan dibanding HTTP/1.1 dalam hal performa dan efisiensi. HTTP/2 mendukung multiplexing, sehingga banyak permintaan dapat berjalan di satu koneksi TCP tanpa blocking. Ini mengurangi latensi secara signifikan. Selain itu, dukungan header compression dan aliran data bidirectional membuat komunikasi lebih ringan dan cepat dibanding REST yang menggunakan HTTP/1.1. Namun, HTTP/2 lebih kompleks dan tidak selalu didukung penuh di semua stack jaringan. Dibandingkan WebSocket, HTTP/2 lebih cocok untuk komunikasi RPC berbasis skema, sedangkan WebSocket unggul dalam pengiriman data real-time event-driven yang tidak terstruktur.

>How does the request-response model of REST APIs contrast with the bidirectional streaming capabilities of gRPC in terms of real-time communication and responsiveness?

Model request-response REST bersifat sinkron dan terbatas pada satu permintaan dan satu respons, yang membuatnya kurang ideal untuk komunikasi real-time dan cocok untuk operasi CRUD biasa. Sebaliknya, gRPC mendukung bidirectional streaming yang memungkinkan klien dan server untuk saling bertukar data secara bersamaan dan kontinu selama koneksi aktif. Hal ini sangat meningkatkan responsivitas dan efisiensi dalam aplikasi seperti live feed, chat, atau game multiplayer. Sementara REST memerlukan polling atau WebSocket untuk mencapai hal serupa, gRPC memberikan fitur ini secara native dengan performa yang lebih baik dan overhead lebih rendah.

>What are the implications of the schema-based approach of gRPC, using Protocol Buffers, compared to the more flexible, schema-less nature of JSON in REST API payloads?

Schema-based approach di gRPC dengan Protobuf menjamin konsistensi data dan kontrak antara client dan server sejak awal. Ini memudahkan validasi otomatis dan deteksi perubahan API secara dini, meningkatkan reliabilitas integrasi antar komponen. Sebaliknya, JSON bersifat lebih fleksibel dan tidak memerlukan schema eksplisit serta mudah dibaca manusia, membuatnya cocok untuk prototyping atau sistem dengan struktur data dinamis. Namun, fleksibilitas ini juga meningkatkan risiko inkonsistensi dan kesalahan parsing. Dengan Protobuf, ukuran pesan lebih kecil dan parsing lebih cepat, tetapi pengembangan membutuhkan tool dan proses kompilasi tambahan, sedangkan JSON bisa langsung digunakan dengan mudah namun lebih boros bandwidth dan CPU.
