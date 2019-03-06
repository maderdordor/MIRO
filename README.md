 # [MiRo Training](https://emarolab.github.io/MiRo-training/)

## Tentang Proyek
Proyek ini dikembangkan untuk mata kuliah Social Robotics pada program magister Teknik Robotika di Universitas Genoa.

### Robot Kompasian MiRo
MiRo merupakan robot berpenampilan menyerupai hewan yang dirancang sebagai prototipe companion.
Robot ini memiliki arsitektur terinspirasi dari biologi berdasarkan pengetahuan neurosains otak mamalia.

### Tujuan Utama
Tujuan dari proyek ini adalah membangun arsitektur perangkat lunak untuk berinteraksi dengan MiRo menggunakan perintah suara dan gestur.
Perhatian robot diperoleh melalui perintah aktivasi suara **"Miro"**.
Hanya setelah perintah aktivasi diberikan, robot dapat mengeksekusi perintah berikutnya.
Kemungkinan perintah yang tersedia adalah:
* **"Good"** - Robot menunjukkan perilaku riang dan berisik, serta memerlukan sentuhan dari pengguna untuk menenangkannya.
* **"Bad"** - Robot menjadi kesal karena dimarahi dan berpaling dari pengguna.
* **"Let's go out"** - Robot meninggalkan stasiun pengisian ke pengguna yang dapat mengontrol gerakan tubuhnya dengan gestur. Gestur yang digunakan dirangkum [di sini](https://ibb.co/6nLxgjw)
* **"Play"** - Robot "mengikuti" pergerakan bola merah.
* **"Kill"** - Robot menjadi marah. Lampu robot menyala merah dan menghasilkan suara bajak laut.
* **"Sleep"** - Robot masuk ke mode istirahat. Perintah aktivasi dinonaktifkan. Oleh karena itu, robot tidak dapat mengeksekusi perintah lain sampai perintah suara baru "Miro" membangunkannya.

## [Arsitektur Perangkat Lunak](https://ibb.co/4MggTyw)
Arsitektur perangkat lunak dapat dilihat dengan mengklik judul bagian.
Arsitektur ini menampilkan node ROS (blok) dan rostopic (biru) yang digunakan node untuk berkomunikasi.

Setiap modul bagian dari arsitektur telah diimplementasikan sebagai node ROS. Untuk memungkinkan komunikasi antar node, digunakan pola messaging Publish/Subscribe.
Arsitektur telah diorganisir agar bersifat modular, skalabel, dan dapat digunakan kembali dalam beberapa langkah.
Karakteristik setiap node akan dianalisis lebih mendalam di bagian berikut.

### Deskripsi Modul

* **command_recognition.py**: node ini menangani perintah suara dan memutuskan perintah mana yang harus dieksekusi oleh MiRo, dengan mempublikasikan ke topik */miro/rob01/platform/control*. Agar dapat merespons instruksi, MiRo harus dibangunkan melalui perintah suara "Miro".
* **gbb_miro.py**: node ini dieksekusi dengan perintah suara "Let's go out" dan menangani perilaku berbasis gestur MiRo. Smartwatch mempublikasikan data IMU ke topik *inertial*. Node **imu_data_map.py** membaca nilai akselerasi dari topik */inertial* dan memetakannya menjadi kecepatan linear dan angular, mempublikasikan data yang dihasilkan ke topik */imu_mapping*. Node **gbb_miro.py** mengimplementasikan perilaku berbasis gestur di mana robot mengikuti gestur pengguna. Node ini mempublikasikan kecepatan linear dan angular ke topik */gbb*.
* **play.py**: node ini dieksekusi dengan perintah suara "Play" dan mempublikasikan ke topik *\miro_play*. MiRo merespons dengan mengikuti bola merah yang ditunjukkan pengguna. Kamera kanan dan kiri MiRo digunakan untuk deteksi bola. Node **hsv_color_filter** melakukan segmentasi warna dari gambar yang diperoleh dari kamera dan mempublikasikan ke */right-left/hsv_color_filter/image*. Gambar tersegmentasi diperoleh oleh **hough_circles** yang mendeteksi objek bulat, menekankan dengan lingkaran merah. Jika bola merah terdeteksi di kamera kanan/kiri, MiRo mulai berputar ke kanan/kiri, sampai objek berada di kedua kamera. Pada titik ini robot mulai bergerak menuju bola dan berhenti saat mencapainya.
* **bad.py**: node ini dieksekusi dengan perintah suara "Bad" dan mempublikasikan ke topik */miro_bad*. MiRo merespons teguran dengan perilaku sedih dan tersinggung. Robot memiringkan dan menundukkan kepala, memutar telinga, dan berputar ke kiri, menunjukkan punggung ke pengguna. Di akhir aksi, lampu LED menyala merah, mengekspresikan kondisi kesalnya.
* **kill.py**: node ini dieksekusi dengan perintah suara "Kill" dan mempublikasikan ke topik *\miro_kill*. MiRo merespons dengan perilaku marah di mana robot menyala merah dan memancarkan suara bajak laut.
* **good.py**: node ini dieksekusi dengan perintah suara "Good" dan mempublikasikan ke topik *\miro_good*. MiRo mencoba menarik perhatian pengguna dengan mengangkat kepalanya dan menggerakan ekornya. Dalam mode ini pengguna dapat menyentuh MiRo di kepala atau tubuh, menghasilkan dua perilaku berbeda yang bergantung pada nilai sensor yang dipublikasikan oleh topik */miro_rob01_platform_sensors*. Ketika sensor kepala diaktifkan, robot berhenti menggerakkan ekor, menundukkan dan memiringkan kepala, menyipitkan mata dan lampu LED menyala pink. Ketika sensor tubuh diaktifkan, robot berhenti menggerakkan ekor, mengangkat kepala, memutar telinga dan lampu LED menyala oranye.
* **sleep.py**: node ini dieksekusi dengan perintah suara "Sleep" dan mempublikasikan ke topik *\miro_sleep*. MiRo merespons dengan masuk ke mode istirahat. Robot menundukkan dan memiringkan kepala dan ekornya, menutup mata dan lampu LED menyala aquamarine. Dalam mode ini perintah suara lain tidak berpengaruh, kecuali diaktifkan dengan perintah suara baru "MiRo"

# Memulai

## Prasyarat

### ROS
Proyek ini dikembangkan menggunakan [ROS](http://wiki.ros.org/kinetic/Installation/Ubuntu):
* rosdistro: kinetic
* rosversion: 1.12.13

### Setup Workstation MiRo
Unduh [Miro Developer Kit](http://labs.consequentialrobotics.com/miro/mdk/).

Ikuti instruksi dari Consequential Robotics [Miro: Prepare Workstation](https://consequential.bitbucket.io/Developer_Preparation_Prepare_workstation.html) untuk mengatur workstation Anda agar dapat bekerja dengan robot ini.
Ikuti dengan ketat instruksi di bagian Install **mdk** karena langkah-langkah selanjutnya bergantung pada ini.
Tidak perlu membuat IP statis untuk workstation Anda (laptop) saat mengatur koneksi dengan MiRo.
Untuk tutorial langkah demi langkah yang jelas, kunjungi [Emarolab Miro Repository](https://github.com/EmaroLab/MIRO.git).

### Perangkat Wearable
Untuk berinteraksi dengan MiRo melalui gestur, digunakan smartwatch dengan sensor IMU 9-sumbu.
[LG G WATCH R](https://www.lg.com/wearable-technology/lg-G-Watch-R-W110)
Ikuti instruksi yang dilaporkan di [imu_stream](https://github.com/EmaroLab/imu_stream) untuk mengunduh aplikasi untuk smartphone dan smartwatch.

### Setup Smartwatch dan Smartphone
Untuk mempublikasikan data sensor IMU dari smartwatch ke node ROS, Anda harus memiliki smartwatch yang terhubung dengan smartphone.
Smartphone bertindak sebagai jembatan antara smartwatch dan master ROS yang berjalan di komputer Anda.

### MQTT ROS Bridge

Untuk berhasil berlangganan topik MQTT dan mempublikasikan konten pesan MQTT ke ROS, ikuti instruksi di [mqtt_ros_bridge](https://github.com/EmaroLab/mqtt_ros_bridge/tree/feature/multiple_smartwatches).
Agar dapat bekerja dengan proyek saat ini, beberapa parameter harus dimodifikasi di imu_bridge.launch
Parameter device_name harus diubah dengan nama smartwatch Anda.

### Antarmuka Suara Berbasis ROS

Untuk berinteraksi secara suara dengan robot, kami menggunakan repositori yang berisi contoh penggunaan antarmuka web untuk berbicara dengan robot. Ini berbasis pada Google Speech Demo untuk melakukan speech-to-text. Kami menonaktifkan fungsionalitas text-to-speech.

Untuk proyek ini, kami menggunakan mikrofon di [LOGITECH Wireless Headset H600](https://www.logitech.com/it-it/product/wireless-headset-h600), tetapi mikrofon apa pun yang terhubung ke laptop Anda seharusnya berfungsi dengan baik.

Buat catkin workspace dan clone semua paket ke folder src

```
$ git clone https://github.com/EmaroLab/ros_verbal_interaction_node.git

```

Untuk informasi lebih lanjut, ikuti instruksi yang terkandung dalam repositori [ros_verbal_interaction_node](https://github.com/EmaroLab/ros_verbal_interaction_node).

### Aplikasi OpenCV

Gambar streaming dari kamera MiRo diproses menggunakan paket [opencv_apps](http://wiki.ros.org/opencv_apps).
Frame kamera mengalami segmentasi warna dan deteksi hough circles.
Untuk menginstalnya:
```
$ git clone sudo apt install ros-kinetic-opencv-apps

```

### MiRo Training

Buat catkin workspace dan clone semua paket ke folder src

```
$ git clone https://github.com/EmaroLab/MiRo-training.git
$ catkin_make
$ source devel/setup.bash
```

## Menjalankan Proyek

Buka terminal baru dan jalankan

```
$ roscore
```
mosquitto harus berjalan di PC Anda agar bridge berfungsi.

Di terminal baru
```
$ mosquitto
```
Pastikan IP di aplikasi IMU_stream di smartphone sama dengan yang ditampilkan oleh

```
$ ifconfig
```

Buka aplikasi IMU_stream di smartwatch
Untuk menguji apakah koneksi antara smartwatch dan ROS berfungsi, mulai transmit data dari aplikasi IMU_stream di smartwatch dan periksa di terminal baru
```
$ rostopic echo \inertial
```
Seharusnya terlihat data IMU yang dipublikasikan oleh smartwatch.

Hubungkan robot MiRo ke ROS Master

```
$ ssh root@<MIRO-IP>
$ sudo nano ./profile
```
Masukkan IP Anda setelah ROS_MASTER_IP

Untuk instruksi lebih detail, lihat [MIRO: Commission MIRO](https://consequential.bitbucket.io/Developer_Preparation_Commission_MIRO.html)

Buka catkin_ws Anda di terminal baru
Perintah berikut akan memulai proyek

```
$ roslaunch command_handler command_handler.launch

```

Parameter yang dapat diubah langsung dari file launch:
* Robot --> sim01 atau rob01 (Nilai default)
	Untuk beralih dari robot nyata ke simulasi (Anda harus launch Gazebo dengan miro sim seperti dijelaskan di [MIRO: Consequential Robotics](https://consequential.bitbucket.io/Developer_Preparation_Commission_MIRO.html)
* Rate node --> 200 Hz (Nilai default)
* Warna bola yang akan dideteksi -->
	h_limit_min = 0; h_limit_max = 10; s_limit_min = 255; s_limit_max = 150; v_limit_min = 255; v_limit_max = 50;
	Ubah nilai min dan max HSV untuk mendeteksi warna berbeda.
	Untuk menemukan nilai HSV warna favorit Anda, periksa [ini](https://alloyui.com/examples/color-picker/hsv).

## Hasil

Klik gambar di bawah untuk video demonstrasi.

[![MiRo-Training - SoRo](https://img.youtube.com/vi/DoKFgs3enpU/0.jpg)](https://www.youtube.com/watch?v=DoKFgs3enpU&feature=youtu.be).

Setiap partisipan diminta mengisi [kuesioner](http://bit.ly/MiroTrainingSurvey) untuk mengevaluasi interaksi dengan robot.

## Rekomendasi
1) Setelah clone repositori [ros_verbal_interaction_node](https://github.com/EmaroLab/ros_verbal_interaction_node), pastikan untuk unsubscribe dari topik */text_to_speech*. Kami sangat memodifikasi file speech_web_interface.html dengan mempublikasikan ke topik yang tidak di-subscribed daripada ke */text_to_speech*. Ini untuk menghindari antarmuka web mengulang apa yang baru dikatakan oleh pengguna.

## Ucapan Terima Kasih

* [mqtt_ros_bridge](https://github.com/EmaroLab/mqtt_ros_bridge)
* [imu_stream](https://github.com/EmaroLab/imu_stream)

### Tim
* Roberta Delrio *roberta.delrio@studio.unibo.it*
* Valentina Pericu *valentina.pericu.@gmail.com*
