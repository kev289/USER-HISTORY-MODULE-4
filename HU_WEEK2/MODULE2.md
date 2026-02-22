
# MODULO 4 SEMANA 2

## BASE DE DATOS UNIVERSIDAD

```sql

##CREACION DE LA BASE DE DATOS
-- CREATE DATABASE gestion_academica_universidad;

##CREACION DE TABLAS
-- Tabla: Docentes
CREATE TABLE docentes (
    id_docente SERIAL PRIMARY KEY,
    nombre_completo VARCHAR(100) NOT NULL,
    correo_institucional VARCHAR(100) UNIQUE NOT NULL,
    departamento_academico VARCHAR(50),
    anios_experiencia INT CHECK (anios_experiencia >= 0)
);

-- Tabla: Estudiantes
CREATE TABLE estudiantes (
    id_estudiante SERIAL PRIMARY KEY,
    nombre_completo VARCHAR(100) NOT NULL,
    correo_electronico VARCHAR(100) UNIQUE NOT NULL,
    genero CHAR(1) CHECK (genero IN ('M', 'F', 'O')),
    identificacion VARCHAR(20) UNIQUE NOT NULL,
    carrera VARCHAR(50) NOT NULL,
    fecha_nacimiento DATE,
    fecha_ingreso DATE DEFAULT CURRENT_DATE
);

-- Tabla: Cursos
CREATE TABLE cursos (
    id_curso SERIAL PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    codigo VARCHAR(10) UNIQUE NOT NULL,
    creditos INT CHECK (creditos > 0 AND creditos <= 6),
    semestre INT CHECK (semestre BETWEEN 1 AND 10),
    id_docente INT,
    CONSTRAINT fk_docente FOREIGN KEY (id_docente) 
        REFERENCES docentes(id_docente) ON DELETE SET NULL
);

-- Tabla: Inscripciones
CREATE TABLE inscripciones (
    id_inscripcion SERIAL PRIMARY KEY,
    id_estudiante INT NOT NULL,
    id_curso INT NOT NULL,
    fecha_inscripcion DATE DEFAULT CURRENT_DATE,
    calificacion_final DECIMAL(3,2) CHECK (calificacion_final BETWEEN 0 AND 5),
    CONSTRAINT fk_estudiante FOREIGN KEY (id_estudiante) REFERENCES estudiantes(id_estudiante) ON DELETE CASCADE,
    CONSTRAINT fk_curso FOREIGN KEY (id_curso) REFERENCES cursos(id_curso) ON DELETE CASCADE
);

##INSERCION DE DATOS

-- Insertar Docentes
INSERT INTO docentes (nombre_completo, correo_institucional, departamento_academico, anios_experiencia) VALUES
('Roberto Gomez', 'roberto@gmail.edu', 'Ingeniería', 12),
('Maria Lopez', 'maria.l@gmail.edu', 'Ciencias Exactas', 4),
('Carlos Ruiz', 'carlos.r@gmail.edu', 'Humanidades', 8);

-- Insertar Estudiantes
INSERT INTO estudiantes (nombre_completo, correo_electronico, genero, identificacion, carrera) VALUES
('Ana Saenz', 'ana@mail.com', 'F', '1010', 'Sistemas'),
('Luis Paez', 'luis@mail.com', 'M', '2020', 'Sistemas'),
('Elena Mar', 'elena@mail.com', 'F', '3030', 'Matemáticas'),
('Jorge Gil', 'jorge@mail.com', 'M', '4040', 'Filosofía'),
('Sara Luz', 'sara@mail.com', 'F', '5050', 'Sistemas');

-- Insertar Cursos
INSERT INTO cursos (nombre, codigo, creditos, semestre, id_docente) VALUES
('Bases de Datos I', 'BD101', 4, 3, 1),
('Programación II', 'PROG2', 3, 2, 1),
('Cálculo Integral', 'CAL2', 4, 2, 2),
('Ética Profesional', 'ETI01', 2, 5, 3);

-- Insertar 8 Inscripciones
INSERT INTO inscripciones (id_estudiante, id_curso, calificacion_final) VALUES
(1, 1, 4.5), (1, 2, 3.8), (2, 1, 4.0), (2, 3, 2.5),
(3, 3, 4.8), (4, 4, 3.9), (5, 1, 4.2), (5, 2, 3.5);

##CONSULTAS 

-- 1. Estudiantes con sus inscripciones y cursos
SELECT e.nombre_completo, c.nombre AS curso, i.calificacion_final
FROM estudiantes e
JOIN inscripciones i ON e.id_estudiante = i.id_estudiante
JOIN cursos c ON i.id_curso = c.id_curso;

-- 2. Cursos dictados por docentes con > 5 años de experiencia
SELECT c.nombre, d.nombre_completo, d.anios_experiencia
FROM cursos c
JOIN docentes d ON c.id_docente = d.id_docente
WHERE d.anios_experiencia > 5;

-- 3. Promedio de calificaciones por curso
SELECT c.nombre, ROUND(AVG(i.calificacion_final), 2) AS promedio
FROM cursos c
JOIN inscripciones i ON c.id_curso = i.id_curso
GROUP BY c.nombre;

-- 4. Estudiantes inscritos en más de un curso
SELECT e.nombre_completo, COUNT(i.id_curso) AS total_cursos
FROM estudiantes e
JOIN inscripciones i ON e.id_estudiante = i.id_estudiante
GROUP BY e.nombre_completo
HAVING COUNT(i.id_curso) > 1;

-- 5. Agregar nueva columna de estado
ALTER TABLE estudiantes ADD COLUMN estado_academico VARCHAR(20) DEFAULT 'Activo';

-- 6. Borrar docente y verificar efecto (ON DELETE SET NULL)
DELETE FROM docentes WHERE id_docente = 2;

-- 7. Cursos con más de 2 estudiantes
SELECT c.nombre, COUNT(i.id_estudiante) AS inscritos
FROM cursos c
JOIN inscripciones i ON c.id_curso = i.id_curso
GROUP BY c.nombre
HAVING COUNT(i.id_estudiante) > 2;

##SUBCONSULTAS 

-- Estudiantes con promedio superior al promedio general
SELECT nombre_completo 
FROM estudiantes 
WHERE id_estudiante IN (
    SELECT id_estudiante FROM inscripciones 
    GROUP BY id_estudiante 
    HAVING AVG(calificacion_final) > (SELECT AVG(calificacion_final) FROM inscripciones)
);

-- Carreras con estudiantes en semestre >= 2 (Usando EXISTS)
SELECT DISTINCT carrera 
FROM estudiantes e
WHERE EXISTS (
    SELECT 1 FROM inscripciones i 
    JOIN cursos c ON i.id_curso = c.id_curso 
    WHERE i.id_estudiante = e.id_estudiante AND c.semestre >= 2
);

-- Estadísticas globales
SELECT 
    COUNT(*) AS total_registros,
    MAX(calificacion_final) AS nota_maxima,
    MIN(calificacion_final) AS nota_minima,
    SUM(creditos) AS total_creditos_inscritos
FROM inscripciones i
JOIN cursos c ON i.id_curso = c.id_curso;

##CREACION DE UNA VISTA

-- Crear la vista 
CREATE VIEW vista_historial_academico AS
SELECT 
    e.nombre_completo AS estudiante,
    c.nombre AS curso,
    d.nombre_completo AS docente,
    c.semestre,
    i.calificacion_final
FROM inscripciones i
JOIN estudiantes e ON i.id_estudiante = e.id_estudiante
JOIN cursos c ON i.id_curso = c.id_curso
LEFT JOIN docentes d ON c.id_docente = d.id_docente;

-- Gestión de Accesos
CREATE ROLE revisor_academico;
GRANT SELECT ON vista_historial_academico TO revisor_academico;
REVOKE ALL ON inscripciones FROM revisor_academico;

##CONTROL DE TRANSACCIONES


--PERDON POR LA FECHA DE ENTREGA ESTOY A LA ESPERA DE EL TALLER MUCHAS GRACIAS ANDRES 

###KEVIN URIBE A.