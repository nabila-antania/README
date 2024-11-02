# README
# Modul Praktikum: Web Service REST untuk Manajemen Buku

## Tujuan
Membuat dan menguji web service REST untuk manajemen buku menggunakan PHP dan MySQL.

## Alat yang Dibutuhkan
1. XAMPP (atau server web lain dengan PHP dan MySQL)
2. Text editor (misalnya Visual Studio Code, Notepad++, dll)
3. Postman

## Langkah-langkah Praktikum

### 1. Persiapan Lingkungan
1. Instal XAMPP jika belum ada.
2. Buat folder baru bernama `movies` di dalam direktori `htdocs` XAMPP Anda.

### 2. Membuat Database
1. Buka phpMyAdmin (http://localhost/phpmyadmin)
2. Buat database baru bernama `moviestore`
3. Pilih database `moviestore`, lalu buka tab SQL
4. Jalankan query SQL berikut untuk membuat tabel dan menambahkan data sampel:

```sql
CREATE TABLE movies (
    id INT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    genre VARCHAR(255) NOT NULL,
    year INT(11) NOT NULL,
    rating INT(11) NOT NULL
);

INSERT INTO movies (title, genre, year, rating) VALUES
('Perewangan', 'Horor', '2024', '8'),
('Absolution', 'Action', '2024', '6'),
('Dilan 1990', 'Roman', '2018', '9'),
('Asih', 'Horor', '2018', '7'),
('Jailangkung', 'Horor', '2017', '9');
```

### 3. Membuat File PHP untuk Web Service
1. Buka text editor Anda.
2. Buat file baru dan simpan sebagai `movies` di dalam folder `movies.php`.
3. Salin dan tempel kode berikut ke dalam `movies`:

```php
<?php
header("Content-Type: application/json; charset=UTF-8");
header("Access-Control-Allow-Origin: *");
header("Access-Control-Allow-Methods: GET, POST, PUT, DELETE");
header("Access-Control-Allow-Headers: Content-Type, Access-Control-Allow-Headers, Authorization, X-Requested-With");

$method = $_SERVER['REQUEST_METHOD'];
$request = [];

if (isset($_SERVER['PATH_INFO'])) {
    $request = explode('/', trim($_SERVER['PATH_INFO'], '/'));
}

function getConnection()
{
    $host = 'localhost';
    $db   = 'moviestore';
    $user = 'root';
    $pass = ''; // Ganti dengan password MySQL Anda jika ada
    $charset = 'utf8mb4';

    $dsn = "mysql:host=$host;dbname=$db;charset=$charset";
    $options = [
        PDO::ATTR_ERRMODE            => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
        PDO::ATTR_EMULATE_PREPARES   => false,
    ];
    try {
        return new PDO($dsn, $user, $pass, $options);
    } catch (\PDOException $e) {
        throw new \PDOException($e->getMessage(), (int)$e->getCode());
    }
}

function response($status, $data = NULL)
{
    header("HTTP/1.1 " . $status);
    if ($data) {
        echo json_encode($data);
    }
    exit();
}

$db = getConnection();

switch ($method) {
    case 'GET':
        if (!empty($request) && isset($request[0])) {
            $id = $request[0];
            $stmt = $db->prepare("SELECT * FROM movies WHERE id = ?");
            $stmt->execute([$id]);
            $movies = $stmt->fetch();
            if ($movies) {
                response(200, $movies);
            } else {
                response(404, ["message" => "movies not found"]);
            }
        } else {
            $stmt = $db->query("SELECT * FROM movies");
            $movies = $stmt->fetchAll();
            response(200, $movies);
        }
        break;

    case 'POST':
        $data = json_decode(file_get_contents("php://input"));
        if (!isset($data->title) || !isset($data->genre) || !isset($data->year) || !isset($data->rating)) {
            response(400, ["message" => "Missing required fields"]);
        }
        $sql = "INSERT INTO movies (title, genre, year, rating) VALUES (?, ?, ?, ?)";
        $stmt = $db->prepare($sql);
        if ($stmt->execute([$data->title, $data->genre, $data->year, $data->rating])) {
            response(201, ["message" => "movies created", "id" => $db->lastInsertId()]);
        } else {
            response(500, ["message" => "Failed to create movies"]);
        }
        break;

    case 'PUT':
        if (empty($request) || !isset($request[0])) {
            response(400, ["message" => "movies ID is required"]);
        }
        $id = $request[0];
        $data = json_decode(file_get_contents("php://input"));
        if (!isset($data->title) || !isset($data->genre) || !isset($data->year) || !isset($data->rating)) {
            response(400, ["message" => "Missing required fields"]);
        }
        $sql = "UPDATE movies SET title = ?, genre = ?, year = ?, rating = ? WHERE id = ?";
        $stmt = $db->prepare($sql);
        if ($stmt->execute([$data->title, $data->genre, $data->year, $data->rating, $id])) {
            response(200, ["message" => "movies updated"]);
        } else {
            response(500, ["message" => "Failed to update movies"]);
        }
        break;

    case 'DELETE':
        if (empty($request) || !isset($request[0])) {
            response(400, ["message" => "movies ID is required"]);
        }
        $id = $request[0];
        $sql = "DELETE FROM movies WHERE id = ?";
        $stmt = $db->prepare($sql);
        if ($stmt->execute([$id])) {
            response(200, ["message" => "movies deleted"]);
        } else {
            response(500, ["message" => "Failed to delete movies"]);
        }
        break;

    default:
        response(405, ["message" => "Method not allowed"]);
        break;
}
?>
```

### 4. Pengujian dengan Postman
1. Buka Postman
2. Buat request baru untuk setiap operasi berikut:

#### a. GET All Books
- Method: GET
- URL: `http://localhost/movies/movies.php`
- Klik "Send"

#### b. GET Specific Book
- Method: GET
- URL: `http://localhost/movies/movies.php/1` (untuk buku dengan ID 1)
- Klik "Send"

#### c. POST New Book
- Method: POST
- URL: `http://localhost/movies/movies.php`
- Headers: 
  - Key: Content-Type
  - Value: application/json
- Body:
  - Pilih "raw" dan "JSON"
  - Masukkan:
    ```json
    {
        "title": "KKN Desa Penari",
        "genre": "Horor",
        "year": 2020,
        "rating": 8
    }
    ```
- Klik "Send"

#### d. PUT (Update) Book
- Method: PUT
- URL: `http://localhost/movies/movies.php/6` (asumsikan ID buku baru adalah 6)
- Headers: 
  - Key: Content-Type
  - Value: application/json
- Body:
  - Pilih "raw" dan "JSON"
  - Masukkan:
    ```json
    {
         "title": "KKN Desa Penari 1",
        "genre": "Horor",
        "year": 2023,
        "rating": 7
    }
    ```
- Klik "Send"

#### e. DELETE Book
- Method: DELETE
- URL: `http://localhost/movies/movies.php/6` (untuk menghapus buku dengan ID 6)
- Klik "Send"

### Kesimpulan
Dalam praktikum ini, Anda telah berhasil membuat web service REST untuk manajemen buku menggunakan PHP dan MySQL. Anda juga telah belajar cara menguji API menggunakan Postman. Praktik ini memberikan dasar yang kuat untuk pengembangan API RESTful lebih lanjut.
### hasil 
![Screenshot (18)](https://github.com/user-attachments/assets/f9dd812e-a6a6-4ba3-933b-64adf4a47ce0) (get)
![Screenshot (19)](https://github.com/user-attachments/assets/000df002-73f9-46c7-a1c4-30f25df794ef) (get1)
![Screenshot (21)](https://github.com/user-attachments/assets/d8163455-b0d2-4244-b311-d29cba4b1b4a) (post)
![Screenshot (22)](https://github.com/user-attachments/assets/91c6345d-ef84-4f81-a97a-a2ff606ed3ba) (put)
![Screenshot (23)](https://github.com/user-attachments/assets/e2aca550-7fbb-45ed-9fc9-dc9e71b876ac) (delet)
