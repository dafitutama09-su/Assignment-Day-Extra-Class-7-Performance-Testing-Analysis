\# Performance Testing Analysis  

\*\*Target Website:\*\* https://resonance.dibimbing.id  

\*\*Tools:\*\* Apache JMeter  



---



\## Overview



Performance Testing Analysis dilakukan untuk mengevaluasi kecepatan, stabilitas, dan skalabilitas sistem dalam menangani beban pengguna secara bersamaan.  

Pengujian ini menggunakan Apache JMeter dengan skenario Load Testing dan dianalisis berdasarkan metrik utama:

\- Response Time

\- Throughput

\- Latency

\- Error Rate



Analisis mengacu pada hasil Summary Report dan View Results Tree



---



\## Test Configuration



\- \*\*Number of Users (Thread):\*\* 50

\- \*\*Ramp-Up Period:\*\* 2 detik

\- \*\*Loop Count:\*\* 1

\- \*\*Test Type:\*\* Load Testing



Konfigurasi ini mensimulasikan lonjakan akses user secara hampir bersamaan.



---



\## Endpoint Analysis



\### 1 Login User



\- \*\*Method:\*\* POST  

\- \*\*Endpoint:\*\* `/api/auth/callback/login`  

\- \*\*Samples:\*\* 50  



\#### Summary Report Result

\- \*\*Average Response Time:\*\* 1601 ms  

\- \*\*Min Response Time:\*\* 539 ms  

\- \*\*Max Response Time:\*\* 3672 ms  

\- \*\*Standard Deviation:\*\* 1199.67 ms  

\- \*\*Error Rate:\*\* 0.00%  

\- \*\*Throughput:\*\* 10.2 request/second  



\#### Analysis

\- Rata-rata response time masih berada di bawah standar umum performa aplikasi (< 3 detik).

\- Terdapat lonjakan response time maksimum hingga >3 detik, menandakan variasi performa saat load meningkat.

\- Error rate 0% menunjukkan proses login berjalan dengan baik.



\#### Conclusion

Endpoint login memiliki performa cukup baik dan stabil, namun perlu diperhatikan karena merupakan critical endpoint.



---



\### 2 Get My Ticket



\- \*\*Method:\*\* GET  

\- \*\*Endpoint:\*\* `/api/trpc/ticket.countMyTickets,ticket.myTicket`  

\- \*\*Samples:\*\* 50  



\#### Summary Report Result

\- \*\*Average Response Time:\*\* 1143 ms  

\- \*\*Min Response Time:\*\* 202 ms  

\- \*\*Max Response Time:\*\* 4177 ms  

\- \*\*Standard Deviation:\*\* 1448.24 ms  

\- \*\*Error Rate:\*\* 10.00%  

\- \*\*Throughput:\*\* 11.0 request/second  



\#### Error Detail

```



HTTP Status: 500 - Internal Server Error

Error querying the database:

FATAL: remaining connection slots are reserved for roles with the SUPERUSER attribute



```



\#### Analysis

\- Response time rata-rata masih baik, namun lonjakan maksimum menunjukkan delay pada sebagian request.

\- Error rate 10% disebabkan oleh database connection exhaustion.

\- Error bukan berasal dari client, melainkan keterbatasan resource database.



\#### Conclusion

Endpoint Get My Ticket tidak stabil pada kondisi concurrent user tinggi dan mulai menunjukkan bottleneck pada sisi database.



---



\### 3 Create New Ticket



\- \*\*Method:\*\* POST  

\- \*\*Endpoint:\*\* `/api/trpc/ticket.createTicket`  

\- \*\*Samples:\*\* 50  



\#### Summary Report Result

\- \*\*Average Response Time:\*\* 2633 ms  

\- \*\*Min Response Time:\*\* 204 ms  

\- \*\*Max Response Time:\*\* 5961 ms  

\- \*\*Standard Deviation:\*\* 1735.16 ms  

\- \*\*Error Rate:\*\* 76.00%  

\- \*\*Throughput:\*\* 3.97 request/second  



\#### Error Detail

```



HTTP Status: 500 - Internal Server Error

Error querying the database:

FATAL: remaining connection slots are reserved for roles with the SUPERUSER attribute



```



\#### Analysis

\- Rata-rata response time mendekati batas toleransi performa.

\- Lonjakan maksimum hingga hampir 6 detik menunjukkan delay berat.

\- Error rate 76% menandakan mayoritas request gagal diproses.

\- Error disebabkan oleh \*\*habisnya koneksi database\*\* saat operasi write.



\#### Conclusion

Endpoint Create New Ticket gagal menangani load tinggi dan menjadi bottleneck paling kritis dalam sistem.



---



\## 4. Latency Analysis



berdasarkan peningkatan response time maksimum dan error HTTP 500 akibat database connection exhaustion, dapat disimpulkan bahwa latency meningkat signifikan pada sisi server khususnya saat backend mencoba mengakses database di kondisi peak load.



---



\## 5. Overall Findings



\- Login endpoint relatif stabil.

\- Get My Ticket mulai mengalami error akibat keterbatasan koneksi database.

\- Create New Ticket mengalami kegagalan masif dan menjadi bottleneck utama.

\- Permasalahan utama sistem berada pada database connection management.



---



\## 6. Recommendations



\### Technical Recommendations

\- Implementasi dan optimasi database connection pooling.

\- Penyesuaian konfigurasi max\_connections database.

\- Optimasi query dan logic backend.

\- Implementasi caching untuk endpoint GET.

\- Pertimbangkan asynchronous processing untuk operasi write.



\### Testing Recommendations

\- Jalankan load testing dengan variasi user (100–200 user).

\- Gunakan ramp-up yang lebih bertahap.

\- Tambahkan monitoring database selama pengujian.



---



\## 7. Final Conclusion



Berdasarkan hasil performance testing dan analisis, sistem mampu menangani beban normal namun belum siap menghadapi concurrent user tinggi.  

Permasalahan utama berasal dari keterbatasan koneksi database yang menyebabkan peningkatan latency, error rate tinggi, dan penurunan throughput, terutama pada endpoint Create New Ticket.



---





