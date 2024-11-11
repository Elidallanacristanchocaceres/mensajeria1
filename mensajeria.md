### 1. Cree un procedimiento almacenado que permita insertar un nuevo usuario retornando un mensaje que indique si la inserción fue satisfactoria.

DELIMITER //
CREATE PROCEDURE insertar_usuario (
    IN p_nombre VARCHAR(50),
    IN p_email VARCHAR(100),
    IN p_contrasena VARCHAR(100)
)
BEGIN
    DECLARE msg VARCHAR(100);

    INSERT INTO usuarios (nombre, email, contrasena)
    VALUES (p_nombre, p_email, p_contrasena);

    SET msg = 'Usuario insertado satisfactoriamente';
    SELECT msg AS resultado;
END //
DELIMITER ;

### 2. Cree un procedimiento almacenado que permita eliminar un usuario retornando un mensaje que indique si la inserción fue satisfactoria.

DELIMITER //
CREATE PROCEDURE eliminar_usuario (
    IN p_usuario_id INT
)
BEGIN
    DECLARE msg VARCHAR(100);

    DELETE FROM usuarios WHERE usuario_id = p_usuario_id;

    SET msg = 'Usuario eliminado satisfactoriamente';
    SELECT msg AS resultado;
END //
DELIMITER ;

### 3. Cree un procedimiento almacenado que permita editar un usuario retornando un mensaje que indique si la inserción fue satisfactoria.

DELIMITER //
CREATE PROCEDURE editar_usuario (
    IN p_usuario_id INT,
    IN p_nombre VARCHAR(50),
    IN p_email VARCHAR(100),
    IN p_contrasena VARCHAR(100)
)
BEGIN
    DECLARE msg VARCHAR(100);

    UPDATE usuarios 
    SET nombre = p_nombre, email = p_email, contrasena = p_contrasena
    WHERE usuario_id = p_usuario_id;

    SET msg = 'Usuario editado satisfactoriamente';
    SELECT msg AS resultado;
END //
DELIMITER ;

### 4.Cree un procedimiento almacenado que permita buscar un usuario por su nombre.

DELIMITER //
CREATE PROCEDURE buscar_usuario (
    IN p_nombre VARCHAR(50)
)
BEGIN
    SELECT * FROM usuarios
    WHERE nombre LIKE CONCAT('%', p_nombre, '%');
END //
DELIMITER ;


### conversacion
CREATE TABLE conversaciones (
    conversacion_id INT AUTO_INCREMENT PRIMARY KEY,
    nombre_conversacion VARCHAR(100) NULL,
    fecha_creacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

### 5. Realice una consulta que permita iniciar una conversación.

INSERT INTO conversaciones (nombre_conversacion) VALUES ('Nueva Conversación');

### 6. Realice un procedimiento almacenado que permita agregar un nuevo participante a la conversación y valide si hay capacidad disponible. La cantidad maxima por cada conversación son 5 usuarios.

DELIMITER //
CREATE PROCEDURE agregar_participante (
    IN p_conversacion_id INT,
    IN p_usuario_id INT
)
BEGIN
    DECLARE num_participantes INT;
    DECLARE msg VARCHAR(100);

    -- Validar si hay capacidad en la conversación
    SELECT COUNT(*) INTO num_participantes
    FROM participantes
    WHERE conversacion_id = p_conversacion_id;

    IF num_participantes < 5 THEN
        INSERT INTO participantes (conversacion_id, usuario_id)
        VALUES (p_conversacion_id, p_usuario_id);
        SET msg = 'Participante agregado satisfactoriamente';
    ELSE
        SET msg = 'La conversación ha alcanzado el límite de participantes';
    END IF;

    SELECT msg AS resultado;
END //
DELIMITER ;

### 7.Realice un procedimiento que permita visualizar los mensaje de una conversación.
 
DELIMITER //
CREATE PROCEDURE ver_mensajes_conversacion (
    IN p_conversacion_id INT
)
BEGIN
    SELECT * FROM mensajes
    WHERE conversacion_id = p_conversacion_id
    ORDER BY fecha_envio;
END //
DELIMITER ;

### 8. Procedimiento para enviar un mensaje a una conversación
DELIMITER //
CREATE PROCEDURE enviar_mensaje (
    IN p_conversacion_id INT,
    IN p_remitente_id INT,
    IN p_contenido TEXT
)
BEGIN
    INSERT INTO mensajes (conversacion_id, remitente_id, contenido)
    VALUES (p_conversacion_id, p_remitente_id, p_contenido);
END //
DELIMITER ;

### creacion de tabla para la 9
CREATE TABLE usuarios (
    usuario_id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    contrasena VARCHAR(100) NOT NULL,
    fecha_creacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);


### 9. Modificación de la tabla mensajes para permitir el envío a todos o a un usuario específico
DELIMITER //
CREATE PROCEDURE ver_mensajes_usuario (
    IN p_usuario_id INT
)
BEGIN
    SELECT * FROM mensajes
    WHERE remitente_id = p_usuario_id OR destinatario_id = p_usuario_id
    ORDER BY fecha_envio;
END //
DELIMITER ;

### 11. 11. Creación de la tabla de palabras no deseadas
CREATE TABLE palabras_no_deseadas (
    palabra_id INT AUTO_INCREMENT PRIMARY KEY,
    palabra VARCHAR(50) NOT NULL
);


### 12. Procedimiento para eliminar un usuario que no respeta las normas de buena conversación
DELIMITER //
CREATE PROCEDURE eliminar_usuario_por_comportamiento (
    IN p_conversacion_id INT
)
BEGIN
    DECLARE usuario_ofensivo INT;
    DECLARE cursor_mensajes CURSOR FOR 
        SELECT remitente_id 
        FROM mensajes 
        WHERE conversacion_id = p_conversacion_id;
        
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET usuario_ofensivo = NULL;

    OPEN cursor_mensajes;
    leer_mensaje: LOOP
        FETCH cursor_mensajes INTO usuario_ofensivo;
        IF usuario_ofensivo IS NULL THEN
            LEAVE leer_mensaje;
        END IF;

        IF EXISTS (
            SELECT 1
            FROM mensajes m
            JOIN palabras_no_deseadas p ON m.contenido LIKE CONCAT('%', p.palabra, '%')
            WHERE m.remitente_id = usuario_ofensivo AND m.conversacion_id = p_conversacion_id
        ) THEN
            DELETE FROM usuarios WHERE usuario_id = usuario_ofensivo;
            LEAVE leer_mensaje;
        END IF;
    END LOOP leer_mensaje;

    CLOSE cursor_mensajes;
END //
DELIMITER ;




