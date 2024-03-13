**Module 5**

Via GUI

- Test plan for endpoints ```/highest-gpa```

<img width="1151" alt="gpa before" src="https://github.com/Marsupilamieue/os232/assets/111985990/94107b2c-deca-42d4-8db7-31e069c7baa4">

- Test plan for endpoints ```/all-student-name```

<img width="1151" alt="all student name before" src="https://github.com/Marsupilamieue/os232/assets/111985990/2c8590c1-3055-4ba2-97f5-f695572e7568">

Via command line

- Test plan for endpoints ```/highest-gpa```

<img width="1025" alt="gpa" src="https://github.com/Marsupilamieue/os232/assets/111985990/2353c746-5514-4a9d-a79a-6670521d338f">

- Test plan for endpoints ```/all-student-name```

<img width="1025" alt="Screenshot 2024-03-12 at 03 06 12" src="https://github.com/Marsupilamieue/os232/assets/111985990/1c4322c5-aeee-4f5d-aa1c-f10abf55e8ed">
<br/><br/>

1. After the profiling and performance optimization process is completed, perform a performance test again using JMeter, see the results, and compare with the first measurement. Is there an improvement from JMeter measurements?

- endpoint ```/all-student```

    saya melakukan refactor pada method getAllStudentsWithCourses. Pada method tersebut terdapat line yang membuat request berjalan sangat lambat, yaitu pada line code yang berisi 
    Pada versi originalnya, kode 
    ```java
    studentCourseRepository.findByStudentId(student.getId())
    ```
    melakukan panggilan ke studentCourseRepository.findByStudentId(student.getId()) untuk setiap siswa, yang bisa menjadi mahal jika ada banyak siswa.

    Pendekatan yang lebih efisien adalah dengan mengambil semua data StudentCourse sekaligus mengelompokkannya berdasarkan studentId. Seperti berikut
    ```java
    public List<StudentCourse> getAllStudentsWithCourses() {
    List<Student> students = studentRepository.findAll();
    List<StudentCourse> allStudentCourses = studentCourseRepository.findAll();

    Map<Long, List<StudentCourse>> studentCoursesByStudentId = allStudentCourses.stream()
            .collect(Collectors.groupingBy(sc -> sc.getStudent().getId()));

    List<StudentCourse> studentCourses = new ArrayList<>();
    for (Student student : students) {
        List<StudentCourse> studentCoursesByStudent = studentCoursesByStudentId.get(student.getId());
        if (studentCoursesByStudent != null) {
            for (StudentCourse studentCourseByStudent : studentCoursesByStudent) {
                StudentCourse studentCourse = new StudentCourse();
                studentCourse.setStudent(student);
                studentCourse.setCourse(studentCourseByStudent.getCourse());
                studentCourses.add(studentCourse);
            }
        }
    }
    return studentCourses;
    }
    ```
    
    metode ini hanya akan membuat dua panggilan database, terlepas dari jumlah siswa. Namun, perlu dicatat bahwa pendekatan ini mengasumsikan bahwa jumlah catatan StudentCourse dapat muat dalam memori. Jika ada terlalu banyak data StudentCourse, mungkin perlu menggunakan pendekatan yang berbeda, seperti paginasi atau pemrosesan batch.

    Setelah mengaplikasikan kode tersebut, terdapat perbedaan yang signifikan pada waktu eksekusi. Tadinya waktu eksekusi sekitar 3 menitan dan setelah dilakukan refactor menjadi kurang dari sepuluh detik

- endpoint ```/highest-gpa```

    saya melakukan refactor pada method findStudentWithHighestGpa. Pada method bisa diubah dengan menambahkan method di studentRepository yang langsung mengembalikan siswa dengan GPA tertinggi, seperti berikut

    ```java
    public interface StudentRepository extends  JpaRepository<Student, Long> {
        Optional<Student> findTopByOrderByGpaDesc();
    }
    ```
    dan mengganti method findStudentWithHighestGpa menjadi seperti berikut
    ```java
    public Optional<Student> findStudentWithHighestGpa() {
        return studentRepository.findTopByOrderByGpaDesc();
    }
    ```
    Dengan cara ini, database akan melakukan semua pekerjaan dalam mencari siswa dengan GPA tertinggi, dan hanya perlu mengambil satu siswa dari database, bukan semua siswa.

    Setelah melakukan refactor, waktu eksekusi turun dari yang tadinya rata-rata-nya ribuan ms menjadi puluhan ms

- endpoint ```/allStudentName```

    saya melakukan refactor pada method findStudentWithHighestGpa. Pada method tersebut terdapat line yang membuat request berjalan sangat lambat, yaitu pada line code yang berisi 
    ```java
    for (Student student : students) {
            result += student.getName() + ", ";
        }
    ```
     penggunaan += untuk menggabungkan string dalam loop bisa menjadi sangat lambat, terutama jika jumlah siswa sangat banyak. Ini karena setiap kali Anda menggunakan +=, Java menciptakan objek String baru, yang bisa menjadi mahal dalam hal waktu dan memori.

    Sebagai alternatif, saya menggunakan StringBuilder, yang lebih efisien untuk operasi penggabungan string yang berulang. Berikut adalah kode yang dioptimalkan:

    ```java
    public ResponseEntity<String> allStudentName() {
        String joinedStudentNames = studentService.joinStudentNames();
        return ResponseEntity.ok(joinedStudentNames);
    }

    public String joinStudentNames() {
        List<Student> students = studentRepository.findAll();
        StringBuilder result = new StringBuilder();
        for (Student student : students) {
            result.append(student.getName()).append(", ");
        }
        // Hapus koma dan spasi terakhir
        if (result.length() > 0) {
            result.setLength(result.length() - 2);
        }
        return result.toString();
    }
    ```

    Setelah melakukan refactor tersebut, eksekusi menjadi semakin cepat dengat rata-rata waktu eksekusi yang turun dari puluh ribuan ms menjadi ribuan ms

2. What is the difference between the approach of performance testing with JMeter and profiling with IntelliJ Profiler in the context of optimizing application performance?
    JMeter digunakan untuk pengujian beban untuk mengukur kinerja keseluruhan aplikasi di bawah kondisi beban yang berbeda, sementara IntelliJ Profiler digunakan untuk profiling yang bertujuan untuk mendapatkan informasi detail tentang perilaku runtime aplikasi.

3. How does the profiling process help you in identifying and understanding the weak points in your application?

    Dengan membandingkan hasil dari CPU usage dan juga time execution. 

4. Do you think IntelliJ Profiler is effective in assisting you to analyze and identify bottlenecks in your application code?
    Ya, IntelliJ Profiler bisa sangat efektif dalam membantu menganalisis dan mengidentifikasi bottleneck dalam kode aplikasi Anda. Ini memberikan informasi detail tentang perilaku runtime aplikasi, seperti berapa banyak waktu yang dihabiskan di setiap metode, seberapa sering metod dipanggil, dan berapa banyak memori yang dikonsumsi oleh objek. Informasi ini sangat berharga dalam mengidentifikasi kode yang tidak efisien, kebocoran memori, dan masalah kinerja lainnya pada tingkat kode.

5. What are the main challenges you face when conducting performance testing and profiling, and how do you overcome these challenges?
    Tantangan yang saya alami saat melakukan performance testing dan juga profiling adalah ketika harus melakukan refactoring dari kode yang kita punya. Cara mengatasi masalah tersebut adalah dengan cara melihat bagian kode mana yang menimbulkan masalah dengan melihat line of code mana yang menimbulkan peningkata cpu usage dan juga tingginya execution time dan kemudian mengubahnya dengan bantuan internet atau AI.

6. What are the main benefits you gain from using IntelliJ Profiler for profiling your application code?
    IntelliJ Profiler memberikan informasi kinerja yang detail tentang perilaku runtime aplikasi, seperti penggunaan CPU, penggunaan memori, dan eksekusi thread. Hal ini dapat membantu dalam mengidentifikasi bottleneck, kinerja dan mengoptimalkan kode sesuai kebutuhan.

7. How do you handle situations where the results from profiling with IntelliJ Profiler are not entirely consistent with findings from performance testing using JMeter?
    1. Memeriksa baik data profil dan data pengujian kinerja untuk melihat apakah ada perbedaan yang mencolok atau tidak konsisten. Mungkin ada perbedaan dalam kondisi di mana pengujian dilakukan, seperti beban sistem, jumlah pengguna simultan, atau konfigurasi lainnya.  
    2. Melakukan pengujian ulang untuk memastikan bahwa hasilnya dapat diproduksi ulang. Ini juga bisa membantu dalam mengidentifikasi faktor-faktor yang mungkin telah mempengaruhi hasil pengujian awal. 

8. What strategies do you implement in optimizing application code after analyzing results from performance testing and profiling? How do you ensure the changes you make do not affect the application's functionality?
    1. Menggunakan data dari profil dan pengujian kinerja untuk mengidentifikasi bagian kode yang menyebabkan masalah kinerja. Ini bisa berupa metode tertentu yang memakan waktu lama untuk dieksekusi atau query yang tidak dioptimalkan. 
    2. Memperbaiki algoritma dan struktur data. biasanya, masalah kinerja dapat diatasi dengan menggunakan algoritma atau struktur data yang lebih efisien. 

    Untuk memastikan perubahan tidak memengaruhi fungsionalitas aplikasi adalah melakukan pengujian unit untuk metod dan memastikan method-method tersebut masih memberikan hasil yang diharapkan setelah optimasi. Kemudian, melakukan pengujian integrasi untuk melihat jika perubahan telah merusak bagian sistem secara keseluruhan. Setelah mengoptimalkan, kita bisa menjalankan pengujian regresi untuk memastikan perubahan baru tidak memengaruhi fungsionalitas yang ada. 