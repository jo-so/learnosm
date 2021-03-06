---
layout: doc
title: Konfigurasi Layanan WMS 
permalink: /id/map-design/wms/
redirect_from:
  - /bi/map-design/wms
  - /bi/map-design/wms/
lang: id
category: map-design
---

<!--
this chapter is a draft because it's not a priority and quite complex
-->

Konfigurasi Layanan WMS
========================

Dalam bab ini, kita akan belajar bagaimana melakukan set up WMS (Web Map Service)
yang akan memungkinkan kita mengirimkan gambar peta dengan mudah melalui
internet. Sebelum kita mendalaminya, kita harus melihat pada perbedaan
antara gambar peta dan data peta yang sering kita lakukan. Sederhananya,
kita perlu membuat jelas perbedaan antara gambar raster dan data vektor.

Sejauh ini kita telah bekerja dengan data vektor. Data vektor adalah 
cara terbaik mendeskripsikan semua titik, garis, dan poligon yang 
terdapat dalam file .osm, shapefile, atau database. File ini berisi
data mentah pada hard drive komputer kita, dan ketika kita ingin 
melihat data seperti apa, kita bergantung pada program seperti QGIS 
atau JOSM untuk membaca semua data dan menggambarnya untuk kita. Format 
vektor memungkinkan kita untuk melakukan analisis kompleks pada data.
Hal ini memungkinkan kita untuk berbicara mengenai QGIS dalam menggambar
objek tertentu dengan cara tertentu, dan mengedit bagian informasi
yang ingin kita ubah. Ini adalah cara untuk mengakses, memperbarui, dan
menggunakan data kita, tetapi ini tidak sangat efisien jika kita
ingin mengkomunikasikan informasi ke orang lain.

Ketika kita ingin menyebarkan informasi, kita biasanya akan membuat
beberapa jenis gambar raster. Gambar raster hanya seperti foto. Ini
mungkin berisi banyak arti, namun tidak mungkin bagi kita untuk
menganalisisnya atau mengedit bagian yang berbeda. Ketika Anda melihat
pada peta di [openstreetmap.org](http://www.openstreetmap.org/), Anda
akan melihat sekumpulan gambar raster. Gambar ini harus lebih kecil
ukurannya dari data yang mereka buat, dan mereka dibuat terlihat lebih
bagus. Ini membautnya lebih mudah untuk melihat peta, tetapi tidak ada
cara untuk mengakses berdasarkan data melalui gambar ini.

Dalam bab ini kita akan belajar mengenai WMS, yang menggunakan protokol
HTTP, sistem yang sama digunakan untuk halaman situs, untuk mengirim
gambar peta melalui internet. Gambar mereka sendiri dibuat oleh server
yang membaca data vektor SIG, yang terdapat di database, shapefile, atau
format geodata lainnya. Server WMS sangat baik untuk mengirimkan gambar 
peta raster melalui internet. Tile peta bersama  ini dapat digunakan sebagai
layer raster (gambar) di QGIS, ArcGIS, dan JOSM. 

![]({{site.baseurl}}/images/en/advanced/en_adv_ch5_image19.png)

Dalam bab ini kita akan belajar bagaimana cara menginstal dan melakukan
set up MapServer ([http://www.mapserver.org/](http://www.mapserver.org/)),
platform open-source untuk penerbitan peta, dan menggunakannya sebagai
layanan WMS kita.

Kita akan mengembangkan di tutorial sebelumnya dimana kita menciptakan
database PostGIS dan dimuat dengan data OpenStreetMap. Dalam lampiran 
bab ini, kita juga akan melalui langkah-langkah yan diperlukan untuk
melakukan set up MapServer dengan data OpenStreetMap di Ubuntu.

Bab ini terdiri beberapa langkah:

1.	Instalasi perangkat lunak MapServer
2.	Membuat Mapfile
3.	Mengubah Mapfile
4.	Mengujicoba WMS
5.	Menambahkan Layer WMS di QGIS
6.	Menambahkan Layer WMS di JOSM

Instalasi Perangkat Lunak MapServer
------------------------------------

1.	Sangat mudah untuk menginstal MapServer dan webserver Apache pada
	Windows menggunakan installer MS4W. Anda dapat mendownload installer
	di [http://www.maptools.org/ms4w/](http://www.maptools.org/ms4w/).
	Klik pada tab Downloads dan mendapatkan file.
	![]({{site.baseurl}}/images/en/advanced/en_adv_ch5_image20.png)
	
2.	Unzip arsip dan copy folder ms4w ke dalam drive root Anda (kemungkinan
	besar `C:\`)
   ![]({{site.baseurl}}/images/en/advanced/en_adv_ch5_image05.png)

3.	Buka folder dan klik dua kali pada apache-install. Jika Anda telah
	menjalankan instalasi ini sebelumnya, Anda perlu menjalankan apache-
	uninstall terlebih dahulu.
   ![]({{site.baseurl}}/images/en/advanced/en_adv_ch5_image16.png)

4.	Buka browser web Anda dan pergilah ke `<http://localhost>`. Anda 
	seharusnya melihat halaman seperti ini:
   ![]({{site.baseurl}}/images/en/advanced/en_adv_ch5_image13.png)

5.	Selanjutnya pergilah ke [http:/localhost/cgi-bin/mapserv.exe](http:/localhost/cgi-bin/mapserv.exe).
	Disini Anda seharusnya melihat:
   ![]({{site.baseurl}}/images/en/advanced/en_adv_ch5_image00.png)

6.	Jika Anda melihat halaman ini dengan benar, semuanya berjalan lancar!
	webserver Apache dijalankan dan MapServer bekerja dengan benar.

Dalam kasus ini Anda bertanya-tanya, <http://localhost> ini berarti Anda
ingin browser situs Anda untuk mengakses webserver berjalan secara lokal.
Dengan kata lain, ini meminta halaman situs dari Apache, webserver yang
Anda instal di mesin Anda.

Membuat Mapfile
----------------

Cara MapServer menyajikan file gambar adalah dengan menggunakan Mapfile,
yang menjelaskan banyak hal tentang peta Anda, termasuk data yang ingin
Anda tampilkan, style yang ingin Anda gunakan, dan proyeksi serta informasi
lain. Menulis Mapfile dapat dapat menjadi sedikit rumit, tetapi untungnya
terdapat plugin QGIS yang dapat secara otomatis membuat Mapfile untuk kita.

1.	Buka QGIS dan pergilah ke Plugins -> Fetch Python Plugins.

2.	Carilah plugin MapServer Export dan instal, jika belum terinstal.
   ![]({{site.baseurl}}/images/en/advanced/en_adv_ch5_image10.png)

3.	Sekarang memuat layer PostGIS yang telah Anda buat di bab pertama, 
	dengan pergi ke “Add PostGIS Layer.”
   ![]({{site.baseurl}}/images/en/advanced/en_adv_ch5_image14.png)

4.	Sebelum kita membuat Mapfile kita perlu melakukan satu hal kecil.
	Plugin tidak bekerja dengan beberapa fitur QGIS baru, sehingga
	kita perlu mengubah simbologi setiap layer kita ke “Old Symbology.”
	Klik kanan disetiap layer dan pergilah ke “Properties.”
   ![]({{site.baseurl}}/images/en/advanced/en_adv_ch5_image01.png)

5.	Di tab “Style”  klik “Old Symbology” di pojok kanan atas.
   ![]({{site.baseurl}}/images/en/advanced/en_adv_ch5_image21.png)

6.	Jawab "Yes" dan klik OK.

7.	Ulangi ini di setiap layernya.

8.	Sekarang kita siap membuka plugin tersebut. Pergilah ke Plugins -> MapServer 
	Export... -> MapServer Export. Mungkin juga berada di bawah Web ->
	MapServer Export atau dengan mengklik pada tombol disini:
   ![]({{site.baseurl}}/images/en/advanced/en_adv_ch5_image08.png)

9.	Plugin ini akan membuat Mapfile kita dengan otomatis, tetapi pertama
	kita perlu mengisi sedikit pilihan. Berikan file tersebut sebuah
	nama dan lokasi, seperti C:\test.map. Atur jenis gambar menjadi png
	serta lebar dan tinggi ke 700. Terakhir, atur URL MapServer menjadi
	http://localhost/cgi-bin/mapserv.exe. 
	
	Ketika Anda telah menyelesaikannya klik OK.
   ![]({{site.baseurl}}/images/en/advanced/en_adv_ch5_image15.png)

10.	Anda mungkin ditanya untuk menyimpan proyek Anda. Lakukanlah sekarang.

11.	Sekarang buka situs browser Anda dan pergi ke:

    http://localhost/cgi-bin/mapserv.exe?MAP=C:\test.map&LAYERS=ALL&MODE=MAP

12.	Jika Mapfile Anda ditempat lain ubah bagian tebal ke lokasi file.
	Anda akan melihat kesalahan berikut:
    ![]({{site.baseurl}}/images/en/advanced/en_adv_ch5_image24.png)

	> Anda mungkin juga mendapatkan sebuah kesalahan seperi “loadStyle(): General error
    > message. Invalid WIDTH...” Dalam kasus ini Anda perlu mengubah
	> lebar baris di mapfile dari 0.91 ke 1 atau lebih besar.
	
	Masalahnya adalah Mapfile kita dibuat dari beberapa asumsi, yaitu
	kita akan memiliki beberapa file yang disiapkan dengan simbol dan 
	huruf. Mari menghapus ini dari Mapfile kita secara manual sehingga
	kita dapat melihat semuanya bekerja.

13.	Carilah Mapfile di komputer Anda dan bukalah dengan Notepad.

14.	Komentar pada baris dimulai “FONTSET” dan “SYMBOLSET” dengan
	memasukkan simbol # di depan mereka.
    ![]({{site.baseurl}}/images/en/advanced/en_adv_ch5_image11.png)

15.	Gulir ke bawah file dan komentar yang muncul pada baris dimulai dengan
	“SYMBOL,” karena ini akan menyebabkan sebuah kesalahan terjadi.
    ![]({{site.baseurl}}/images/en/advanced/en_adv_ch5_image06.png)

16.	Simpan file kemudian muat ulang kembali halaman pada browser Anda.
	MapServer sekarang seharusnya me-rendering dan melayani peta Anda
	dengan benar.

![]({{site.baseurl}}/images/en/advanced/en_adv_ch5_image03.png)

Mengubah Mapfile
-----------------

Hal ini mungkin membuat banyak penyesuaian dengan style pada peta Anda
dengan menyesuaikan Mapfile. Cara termudah untuk mengubah ketebalan
dan warna garis mungkin untuk mengubah mereka di QGIS dan kemudian
membuat kembali Mapfile, namun ini juga mudah melakukan penyesuaian
ke file secar langsung.

Jika Anda membuka Mapfile Anda di Notepad, perhatikan bahwa terdapat
banyak informasi di bagian atas. Beberapa mungkin Anda mengenalinya,
tetapi sebagian besar tidak akan mungkin mengenalinya. Anda akan 
mengenali baris yang bertuliskan “SIZE 700 700” karena Anda mengatakan
plugin QGIS membuat gambar 700x700 piksel.

Kami tidak akan membahas Mapfile secara mendalam disini, tetapi jika
Anda menggulir ke bwah file Anda akan melihat empat bagian bahwa menjadi
dengan kata “LAYER” dan diakhiri dengan “END.” Setiap bagian ini membahas
segala sesuatu yang MapServer perlu tahu mengenai salah satu layer yang
membuat peta kita. Layer jalan akan terlihat seperti ini:

    LAYER
     NAME 'planet_osm_roads'
     TYPE LINE
     DUMP true
     TEMPLATE fooOnlyForWMSGetFeatureInfo
    EXTENT 34.121031 31.071647 35.214117 31.691029
      CONNECTIONTYPE postgis
      CONNECTION "dbname='osmgis' user='postgres' password='postgres' sslmode=disable"
      DATA 'way FROM "planet_osm_roads" USING UNIQUE osm_id USING srid=4326'
      METADATA
        'ows_title' 'planet_osm_roads'
      END
      STATUS OFF
      TRANSPARENCY 100
      PROJECTION
      'proj=longlat'
      'datum=WGS84'
      'no_defs'
      END
      CLASS
         NAME 'planet_osm_roads'
         STYLE
           WIDTH 0.91
           COLOR 36 21 207
         END
      END
    END
   
Satu hal yang mungkin dicatat disini adalah blok akhir yang dimulai
dengan “STYLE.” Disini Anda dapat mengubah ketebalan garis pada layer
ini, dan warna garis. Catatan bahwa tiga angka di samping warna untuk
mewakili nilai merah, hijau, dan biru dari warna. Setiap angka dapat 
naik ke 255. Misalnya, 0 0 255 akan menjadi biru, sedangkan 255 0 255
akan menjadi ungu, karena ini adalah campuran merah dan biru.

Juga, Anda mungkin memperhatikan bahwa layer titik kita tidak ditampilkan
seperti yang kita maksud - karena kita telah menghapus simbol dalam Mapfile
kita. Agar layer titik tersebut menampilkan simbol, kita perlu mendefinisikan
file gambar yang ingin kita gunakan. Untuk melakukan ini, kita membuat
sebuah blok di Mapfile kita, diatas bagian LAYER. Blok tersebut akan terlihat 
seperti ini:
   

    SYMBOL
      NAME "circle"
      TYPE PIXMAP
      IMAGE "circle.png"
    END

![circle.png]({{site.baseurl}}/images/en/advanced/en_adv_ch5_image23.png)

Hal ini mendefinisikan sebuah simbol dengan nama "lingkaran" dan 
menghubungkannya dengan gambar circle.png. Ini mengasumsikan bahwa
kita memiliki sebuah simbol dengan nama ini di direktori yang sama.
Kemudian kita dapat menggunakan definisi ini di layer titik kita dan
titik akan di-render dengan simbol. Kita membuang komentar baris "SYMBOL"
dan memuat kembali browser.

    CLASS
       NAME 'planet_osm_point'
       STYLE
         SYMBOL "circle"
         SIZE 7.0
         OUTLINECOLOR 0 0 0
         COLOR 187 154 69
       END
    END

![map rendered with icon for points]({{site.baseurl}}/images/en/advanced/en_adv_ch5_image17.png)

Ada banyak yang Anda dapat lakukan dengan style MapServer. Untuk
informasi lebih lanjut kunjungilah.

Menguji WMS
-------------

Server WMS Anda siap untuk dikonfiguarasi, jadi mari kita mencobanya.
Seperti yang kita sudah lihat, WMS bekerja di atas protokol HTTP, jadi
kita dapat mengunjungi link di browser untuk menguji kemampuan server
tersebut.

1.	Buka browser Anda dan kunjungi:
   [http://localhost/cgi-bin/mapserv.exe?map=C:\\test.map&SERVICE=WMS&VERSION=1.1.1&REQUEST=GetCapabilities](http://localhost/cgi-bin/mapserv.exe?map=C:/test.map&SERVICE=WMS&VERSION=1.1.1&REQUEST=GetCapabilities)
	
	Ini akan menyebabkan file akan terdownload. file tersebut bernama 
	mapserv.exe, tetapu faktanya file teks XML yang menjelaskan kemampuan
	server Anda.

2.	Buka file dengan Notepad dan Anda akan melihat beberapa informasi
	yang dihubungkan dengan server Anda dan Mapfile. Cari file untuk
	kata "error." Semoga itu tidak ada!

Menambah Layer WMS di QGIS
---------------------------

1. Anda dapat menambahkan layer WMS di QGIS dengan tombol “Add WMS Layer”.
   ![]({{site.baseurl}}/images/en/advanced/en_adv_ch5_image04.png)

2. Untuk menambah server WMS Anda, klik "New".

3. Sediakan sebuah nama untuk server Anda dan untuk memasukkan URL:
   [http://localhost/cgi-bin/mapserv.exe?map=C:/test.map](http://localhost/cgi-bin/mapserv.exe?map=C:/test.map)
   ![]({{site.baseurl}}/images/en/advanced/en_adv_ch5_image18.png)

4. Klik OK.

5. Sekarang klik “Connect” untuk melihat layer yang ada. Ini adalah
	empat layer yang didefinisikan di Mapfile.
   ![]({{site.baseurl}}/images/en/advanced/en_adv_ch5_image22.png)

Anda dapat menambahkan beberapa layer ini sebagai layer raster di QGIS.
Ingatlah bahwa layer WMS ini adalah gambar, dan bukan data vektor, jadi
Anda tidak dapat mengeditnya, tetapi ini adalah cara yang bagus untuk
menyediakan referensi gambar.

Menambah Layer WMS di JOSM
--------------------------

Kita juga dapat dengan mudah menambahkan peta WMS kita sebagai layer di JOSM.

1. Buka JOSM dan pergilah ke Menu Preferences.

2. Klik pada tab yang bernama “WMS TMS.”
   ![]({{site.baseurl}}/images/en/advanced/en_adv_ch5_image12.png)

3. Di bagian bawah, klik pada tombol +.

4. Di bawah “Service URL” masukkan
   [http://localhost/cgi-bin/mapserv.exe?map=C:/test.map](http://localhost/cgi-bin/mapserv.exe?map=C:/test.map)
   ![]({{site.baseurl}}/images/en/advanced/en_adv_ch5_image09.png)

5. Klik “Get Layers.” Anda akan melihat daftar empat layer di server WMS Anda.

6. Pilih salah satu dan klik OK dan OK lagi.

7. Setelah Anda mendownload beberapa data dari OSM, Anda akan dapat
	memuat layer WMS Anda. Pergilah ke “Imagery Menu” dan temukan
	layer baru Anda, yang bernama “Unnamed Imagery Layer” (kecuali
	jika Anda mengubahnya)
   ![]({{site.baseurl}}/images/en/advanced/en_adv_ch5_image07.png)

8. Klik pada layer baru Anda untuk menambahkannya sebagai layer latar
	belakang.

Ringkasan
-----------

WMS adalah protokol umum yang digunakan untuk mengirim gambar peta 
melalui internet. Dengan WMS, pengguna membuat permintaan untuk 
gambar peta dengan parameter tertentu. 

Dalam bab ini kita telah mempelajari bagaimana melakukan set up 
MapServer dan mengkonfigurasinya sebagai server WMS, menggunakan
data yang kita import ke dalam PostGIS untuk membuat gambar peta.
Anda dapat mendapatkan informasi yang lebih banyak di arsitektur
WMS di
[http://docs.geoserver.org/latest/en/user/services/wms/reference.html](http://docs.geoserver.org/latest/en/user/services/wms/reference.html).

Di bab selanjutnya kita akan melihat cara lain mendesain dan 
mengirimkan peta di internet. Jika Anda tertarik untuk melakukan
pengaturan MapServer di Linux Ubuntu, lihat Lampiran di akhir bab ini.

Lampiran - Instalasi MapServer di Linux Ubuntu
----------------------------------------------

Di lampiran ini kita akan melalui langkah-langkah menginstal untuk
melakukan pengaturan MapServer di Linux Ubuntu. Langkah-langkah ini
mengkuti Lampiran pada bab satu, dan kita mengasumsi bahwa Anda telah
menginstal PostGIS dan mengimport data OpenStreetMap menggunakan 
osm2pgsql.

**Menginstal MapServer dan Apache**

Untuk menginstal MapServer di Ubuntu 11.04 (Natty Narwhal), jalankan 
perintah ini:

    sudo apt-get -y install apache2
    sudo apt-get -y install python-software-properties
    sudo add-apt-repository ppa:ubuntugis/ubuntugis-unstable
    sudo apt-get update
    sudo apt-get -y install cgi-mapserver mapserver-bin

Untuk menginstal MapServer di Ubuntu >= 11.10:

    sudo apt-get -y install apache2
    sudo apt-get -y install cgi-mapserver mapserver-bin

Jika Anda mengunjungi
[http://localhost/cgi-bin/mapserv](http://localhost/cgi-bin/mapserv), it

harus mengatakan “Tidak ada informasi query untuk memecahkan kode. 
QUERY\_STRING diatur, tetapi kosong.” Catatan bahwa jika Anda mengaksesnya 
dari komputer lain (jika Anda melakukan pengaturan di remote server),
ganti localhost dengan server alamat IP.

**Membuat Mapfile**

Instalasi MapServer selesai, sekarang kita semua membutuhkan Mapfile yang
berisi informasi yang tepat untuk layer kita. Karena kita mengkonfigurasi
MapServer di server Ubuntu, kita tidak dapat menggunakan QGIS disini untuk
membuat Mapfile seperti kita lakukan di Windows. Kita tidak membahas 
kerumitan Mapfile disini, tetapi di bawah ini adalah Mapfile yang akan
bekerja dengan setup yang telah kita buat sejauh ini. Catatan bahwa kita
hanya memiliki informasi untuk satu layer, tetapi Anda dapat menambahkannya
dengan mudah layer tambahan menggunakan format yang sama.

/var/www/test.map

    MAP
      NAME "My-Test-Map"
      # Map image size
      SIZE 700 700
      UNITS meters
   
      EXTENT 3756680.934870 3642952.056250 3899342.315130 3723789.193750
      PROJECTION
        'proj=longlat'
        'datum=WGS84'
        'no_defs'
      END
      
      # Background color for the map canvas -- change as desired
      IMAGECOLOR 255 255 255
      IMAGEQUALITY 95
      IMAGETYPE png
    
      OUTPUTFORMAT
        NAME png
        DRIVER 'GD/PNG'
        MIMETYPE 'image/png'
        IMAGEMODE RGBA
        EXTENSION 'png'
      END
    
      WEB
        IMAGEPATH '/tmp/'
        IMAGEURL '/tmp/'
      
        # WMS server settings
        METADATA
          'ows_title'           'My-Test-Map'
          'ows_onlineresource'  'http://198.61.205.151/cgi-bin/mapserv?MAP=/var/www/test.map'
          'ows_srs'             'EPSG:4326'
        END
    
        TEMPLATE 'fooOnlyForWMSGetFeatureInfo'
      END
    
      LAYER
        NAME 'planet_osm_line'
        TYPE LINE
        DUMP true
        TEMPLATE fooOnlyForWMSGetFeatureInfo
        UNITS METERS
        EXTENT 3756680.934870 3642952.056250 3899342.315130 3723789.193750
        CONNECTIONTYPE postgis
        CONNECTION "dbname='osm' user='postgres' sslmode=disable"
        DATA 'way FROM "planet_osm_line" USING UNIQUE osm_id USING srid=900913'
        METADATA
          'ows_title' 'planet_osm_line'
        END
        STATUS OFF
        TRANSPARENCY 100
        PROJECTION
          'proj=longlat'
          'datum=WGS84'
          'no_defs'
        END
        CLASS
          NAME 'planet_osm_line'
          STYLE
            WIDTH 0.91
            COLOR 46 195 130
          END
        END
      END
    END
    
Perhatikan bahwa Anda perlu mengubah baris “EXTENT” tergantung pada 
lokasi data Anda. Luasan ini disediakan dalam eter, karena data kita
dalam proyeksi mercator.

Anda sekarang dapat mengakses WMS Anda di QGIS dan JOSM. Lihat bagian
6 dan 7 bab ini untuk informasi lebih lanjut. WMS alamat Anda akan menjadi:

http://<YOUR_SERVER_IP>/cgi-bin/mapserv?MAP=/var/www/test.map

<!--
[[a]](#cmnt_ref1)MrPatrickOswald:

hanya pertanyaan sederhana... kenapa menggunakan mapserver dan bukan
geoserver? atau setidaknya menyebutkan bahwa ada juga geoserver dan 
keputusan mengapa Anda memilih mapserver.

[[b]](#cmnt_ref2)MrPatrickOswald:

sebagai contoh mungkin lebih baik atau tambahan menggunakan gambar orbview
(atau kita menggunakan bappeda -diperoleh GeoEye-Data) yang selain dapat
diguanakan di JOSM (tidak melalui plugin import image tetapi melalui 
fungsi layer-WMS..) Saya memiliki pengalaman bahwa plugin import image 
adalah lamban yang jauh lebih baik menggunakan wms untuk menggunakan
citra satelit Anda yang telah digeoreferensi sendiri di JOSM. Saya tetap
masih lebih suka Geoserver karena mudah untuk mengkonfigurasi kebutuhan
ini (latar belakang citra satelit untuk JOSM).

--> 
