# tunggu-db

Composite action untuk menunggu service database atau cache benar-benar siap sebelum tes berjalan.

Di runner Kubernetes (ARC), opsi `--health-cmd` pada `services:` **tidak** dipakai untuk menunggu,
jadi langkah tes bisa mulai sebelum database menerima koneksi. Pakai action ini sebagai langkah
pertama setelah `actions/checkout`.

## Input

| Input | Wajib | Default | Keterangan |
|---|---|---|---|
| `type` | ya | | `mysql`, `mariadb`, `postgres`, atau `redis` |
| `host` | | sama dengan `type` | nama service di `services:` |
| `port` | | 3306 / 5432 / 6379 | |
| `user` | | `root` (MySQL/MariaDB), `postgres` | tidak dipakai untuk Redis |
| `password` | | kosong | tidak pernah dicetak ke log |
| `timeout` | | `300` | detik |

Cara cek: `mysqladmin ping` / `mariadb-admin ping`, `pg_isready`, atau `redis-cli ping` bila klien
tersedia di image (semua image `ghcr.io/webekspres/ci-*` menyediakannya). Tanpa klien, action
memeriksa port TCP lewat bash `/dev/tcp`.

Bila batas waktu habis, job gagal dengan pesan yang menyebut host, port, dan cara cek yang dipakai.

## Contoh

```yaml
jobs:
  test:
    runs-on: arc-ci
    container:
      image: ghcr.io/webekspres/ci-php:8.4
    services:
      mysql:
        image: mysql:8.4
        env: { MYSQL_ROOT_PASSWORD: root, MYSQL_DATABASE: app }
    steps:
      - uses: actions/checkout@v4
      - uses: Webekspres/.github/actions/tunggu-db@main
        with:
          type: mysql
          password: root
      - run: php artisan test
```

Postgres:

```yaml
      - uses: Webekspres/.github/actions/tunggu-db@main
        with:
          type: postgres
          user: app
```
