# Data Engineering Practical Project

Repository for the practical project of the Data Engineering course, taught by Professor André Britto de Carvalho at the Federal University of Sergipe (UFS).

The project is divided into three parts: relational CRUD (PostgreSQL), NoSQL CRUD (MongoDB), and data integration into a star schema (ETL with Apache Hop).

## Students

- Evilyn Aquino dos Santos
- Nicolle Rillary Santana Silva

## Repository structure

```
data-engineering-project/
├── part1-relational-crud/    # CRUD in PostgreSQL (AWS RDS)
├── part2-nosql-crud/         # CRUD in MongoDB (AWS EC2)
├── part3-data-integration/   # Star schema + ETL (Apache Hop)
└── README.md
```

## Data model

The table structure follows the **"universidade"** relational schema worked on in class (dump provided by the professor). This schema has 16 tables in total; the CRUD developed in this project (Parts 1 and 2) specifically handles four of them:

- **usuario** — key: `cpf`. Personal and access data (name, date of birth, email and phone list, login, password)
- **estudante** — key: `mat_estudante`. Specialization of user (references `usuario` via `cpf`; fields `mc` and `ano_ingresso`)
- **curso** — key: `idcurso`. Defined by name, degree type (Bacharelado/Licenciatura Plena), shift, campus, and level
- **vinculo** — key: `idvinculo`. Association between a student and a course over time (status: Ativo/Cancelada/Formando/Graduado; entry and exit dates)

The other 12 tables of the professor's schema (`professor`, `departamento`, `disciplina`, `semestre`, `turma`, `leciona`, `cursa`, `projeto`, `plano`, `sala`, `horario`, `alocacao`) are part of the complete relational database and serve as a data source for Part 3 (ETL / star schema).

---

## Part 1 — Relational CRUD (PostgreSQL)

Complete CRUD in Python, connected to a PostgreSQL instance hosted on AWS RDS.

### Files
| File | Description |
|---|---|
| `dump_completo_professor.sql` | Professor's original script: creates the full `universidade` schema (16 tables) with sample data |
| `db.py` | Database connection module (reads credentials from `.env`) |
| `crud.py` | Pure create, read, update, and delete functions for usuario/curso/estudante/vinculo (16 functions, no user interface) |
| `exibicao.py` | Functions responsible only for formatting and printing data to the screen |
| `menu.py` | Command-line interface (interactive menu) — this is the program the user actually runs |
| `requirements.txt` | Python dependencies |

### How to run

1. Create a `.env` file inside `parte1-crud-relacional/` with:
   ```
   DB_HOST=your-endpoint.rds.amazonaws.com
   DB_PORT=5432
   DB_NAME=database_name
   DB_USER=postgres
   DB_PASSWORD=your_password
   ```
2. Install the dependencies:
   ```bash
   pip install -r requirements.txt
   ```
3. Run `dump_completo_professor.sql` on the database (via pgAdmin 4 or `psql`) — this creates the 16 tables of the `universidade` schema, including the 4 used by the CRUD.
4. Run the interactive program:
   ```bash
   python menu.py
   ```
   This opens a terminal menu where you can choose the entity (user/course/student/link) and the operation (create/search/list/update/delete), typing in the data on the spot.

---

## Part 2 — NoSQL CRUD (MongoDB)

Complete CRUD in Python, connected to a self-managed MongoDB instance hosted on an AWS EC2 server.

### Relational → MongoDB mapping

The mapping covers **all 16 tables** of the professor's schema, represented in 7 collections:

| Relational table(s) | MongoDB representation |
|---|---|
| `usuario` | `usuarios` collection (natural key: `cpf`) |
| `estudante`, `professor` | Subdocuments embedded in `usuarios` (1:1 relationship) |
| `vinculo`, `cursa`, `plano` | Lists embedded within `usuarios.estudante` (1:N relationship) |
| `curso` | Dedicated `cursos` collection |
| `departamento` | Dedicated `departamentos` collection |
| `disciplina` | Dedicated `disciplinas` collection |
| `semestre` | Dedicated `semestres` collection |
| `projeto` | Dedicated `projetos` collection |
| `turma`, `leciona`, `sala`, `horario`, `alocacao` | `turmas` collection, with embedded `professores` and `alocacoes` |

The **CRUD in code** (`crud_mongo.py`) handles only the four entities required by the assignment (user, student, course, link) — the other collections exist only for representation/mapping purposes, with no associated CRUD operations.

### Files
| File | Description |
|---|---|
| `mongo_schema.js` | Script that creates all 7 collections, with schema validation (`$jsonSchema`) and indexes |
| `mongo_db.py` | Database connection module (reads credentials from `.env`) |
| `crud_mongo.py` | Create, read, update, and delete functions for user/student/course/link |
| `requirements_mongo.txt` | Python dependencies |
| `.env.example` | Environment variables template (no real data) |

### How to run

1. Create a `.env` file inside `parte2-crud-nosql/` with:
   ```
   MONGO_URI=mongodb://user:password@your-ec2-address:27017/?authSource=admin
   MONGO_DB_NAME=bancoufs_nosql
   ```
2. Install the dependencies:
   ```bash
   pip install -r requirements_mongo.txt
   ```
3. Run `mongo_schema.js` in MongoDB (via `mongosh` or the MongoDB Compass shell) — it is recommended to paste it in small blocks (one `db.createCollection(...)` at a time).
4. Run the test script:
   ```bash
   python crud_mongo.py
   ```

---

## Part 3 — Data Integration

Construction of a star schema and ETL pipelines with Apache Hop, integrating data from the relational database (Part 1) with CSV files from the [dados.ufs.br](https://dados.ufs.br/group/ensino) portal.

The star schema models undergraduate classes (turmas), linking professor, course subject, department, semester, and campus, with the metrics of enrolled students, average grade, approved, and failed.

### Files
| File | Description |
|---|---|
| `dw_schema.sql` | Script that creates the dimension tables and the fact table |
| `carga_dim_departamento.hpl` | Load pipeline for the Department dimension |
| `carga_dim_professor.hpl` | Load pipeline for the Professor dimension |
| `carga_dim_disciplina.hpl` | Load pipeline for the Subject dimension |
| `carga_dim_semestre.hpl` | Load pipeline for the Semester dimension |
| `carga_dim_campus.hpl` | Load pipeline for the Campus dimension |
| `carga_fato_turma.hpl` | Load pipeline for the Turma fact table |
| `uni-csv-unidades-academicas-da-ufs.csv`, `doc-csv-docentes-da-ufs.csv`, `com-csv-componentes-curriculares-da-ufs.csv`, `tur-csv-turmas-de-2025.csv` | Data sources from the dados.ufs.br portal |

### Data sources
- **Relational (Part 1):** all original rows from the dump, plus the rows inserted via the CRUD.
- **CSV (dados.ufs.br):** Academic Units, Curricular Components, Faculty, and Classes (2019–2025).

### How to run

1. Create a dedicated database on the same RDS server as Part 1 (separate from the relational database) and run `dw_schema.sql`.
2. In Apache Hop, create a database connection named `dw_ufs` pointing to that database.
3. Open the pipelines inside a Hop project whose root is the `parte3-integracao-de-dados/` folder — the input CSVs are referenced by relative path (`${Internal.Entry.Current.Directory}`).
4. Run the dimension pipelines (`carga_dim_*.hpl`) first and `carga_fato_turma.hpl` last.

---

## Security

No `.env` file (containing real credentials) is versioned in this repository — each folder has a `.gitignore` that prevents it from being committed. Use the `.env.example` files as a reference to create your own local copy.
