<?php
// 1. KONFIGURASI & DATABASE
$db_file = 'data_kelulusan.db';
$db = new PDO("sqlite:$db_file");

// Buat table jika belum ada
$db->exec("CREATE TABLE IF NOT EXISTS siswa (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    nisn TEXT,
    nama TEXT,
    status TEXT,
    file_skl TEXT
)");

$message = "";

// 2. LOGIKA UPLOAD (ADMIN)
if (isset($_POST['admin_upload'])) {
    $nisn = $_POST['nisn'];
    $nama = $_POST['nama'];
    $status = $_POST['status'];
    
    // Proses Upload File
    $target_dir = "uploads/";
    if (!file_exists($target_dir)) mkdir($target_dir, 0777, true);
    
    $file_name = time() . "_" . basename($_FILES["file_skl"]["name"]);
    $target_file = $target_dir . $file_name;

    if (move_uploaded_file($_FILES["file_skl"]["tmp_name"], $target_file)) {
        $stmt = $db->prepare("INSERT INTO siswa (nisn, nama, status, file_skl) VALUES (?, ?, ?, ?)");
        $stmt->execute([$nisn, $nama, $status, $file_name]);
        $message = "<div class='alert success'>Data berhasil ditambahkan!</div>";
    }
}

// 3. LOGIKA CEK KELULUSAN (SISWA)
$siswa_found = null;
if (isset($_GET['cek_lulus'])) {
    $nisn_cari = $_GET['nisn_cari'];
    $stmt = $db->prepare("SELECT * FROM siswa WHERE nisn = ?");
    $stmt->execute([$nisn_cari]);
    $siswa_found = $stmt->fetch(PDO::FETCH_ASSOC);
    if (!$siswa_found) $message = "<div class='alert danger'>Data NISN tidak ditemukan!</div>";
}
?>

<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <title>Sistem Kelulusan SMK</title>
    <style>
        body { font-family: sans-serif; background: #f4f7f6; padding: 20px; line-height: 1.6; }
        .container { max-width: 800px; margin: auto; background: white; padding: 20px; border-radius: 8px; box-shadow: 0 2px 10px rgba(0,0,0,0.1); }
        h2 { border-bottom: 2px solid #3498db; padding-bottom: 10px; color: #2c3e50; }
        .section { margin-bottom: 40px; padding: 15px; border: 1px solid #eee; border-radius: 5px; }
        label { display: block; margin-top: 10px; font-weight: bold; }
        input, select { width: 100%; padding: 10px; margin-top: 5px; border: 1px solid #ddd; border-radius: 4px; box-sizing: border-box; }
        button { background: #3498db; color: white; border: none; padding: 10px 20px; margin-top: 15px; cursor: pointer; border-radius: 4px; }
        button:hover { background: #2980b9; }
        .alert { padding: 10px; margin-bottom: 20px; border-radius: 4px; }
        .success { background: #d4edda; color: #155724; }
        .danger { background: #f8d7da; color: #721c24; }
        .result-box { text-align: center; padding: 20px; border: 2px dashed #3498db; margin-top: 20px; }
        .LULUS { color: green; font-weight: bold; font-size: 1.5em; }
        .TIDAK_LULUS { color: red; font-weight: bold; font-size: 1.5em; }
    </style>
</head>
<body>

<div class="container">
    <h2>🎓 Sistem Informasi Kelulusan SMK</h2>
    <?= $message ?>

    <div class="section">
        <h3>Cek Kelulusan</h3>
        <form method="GET">
            <label>Masukkan NISN Anda:</label>
            <input type="text" name="nisn_cari" required placeholder="Contoh: 00123456">
            <button type="submit" name="cek_lulus">Lihat Hasil</button>
        </form>

        <?php if ($siswa_found): ?>
            <div class="result-box">
                <p>Nama: <strong><?= htmlspecialchars($siswa_found['nama']) ?></strong></p>
                <p>Status Kelulusan:</p>
                <div class="<?= $siswa_found['status'] ?>"><?= str_replace('_', ' ', $siswa_found['status']) ?></div>
                <br>
                <?php if ($siswa_found['file_skl']): ?>
                    <a href="uploads/<?= $siswa_found['file_skl'] ?>" target="_blank">
                        <button style="background: #27ae60;">Download SKL (PDF)</button>
                    </a>
                <?php endif; ?>
            </div>
        <?php endif; ?>
    </div>

    <div class="section" style="background: #f9f9f9;">
        <h3>Panel Admin (Input Data Siswa)</h3>
        <form method="POST" enctype="multipart/form-data">
            <label>NISN:</label>
            <input type="text" name="nisn" required>
            
            <label>Nama Siswa:</label>
            <input type="text" name="nama" required>
            
            <label>Status:</label>
            <select name="status">
                <option value="LULUS">LULUS</option>
                <option value="TIDAK_LULUS">TIDAK LULUS</option>
            </select>
            
            <label>Upload File SKL (PDF/JPG):</label>
            <input type="file" name="file_skl" accept=".pdf,.jpg,.jpeg,.png" required>
            
            <button type="submit" name="admin_upload" style="background: #34495e;">Simpan Data Siswa</button>
        </form>
    </div>
</div>

</body>
</html>
