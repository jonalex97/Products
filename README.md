Cómo ejecutar la solución localmente:
- Clonar el repositorio.
- Configurar las variables de entorno en appsettings.Development.json, incluyendo la cadena de conexión a SQL Server y los parámetros de JWT.
- Ejecutar el proyecto API como startup desde Visual Studio 2022 o con el comando dotnet run --project src/Producto.Api.

Requisitos previos:
- .NET 8 SDK
- SQL Server (local o en contenedor)
- Visual Studio 2022 o VS Code
- Entity Framework Core CLI

Descripción general de la arquitectura usada:
El proyecto sigue los principios de Clean Architecture, con separación clara de responsabilidades:
- Producto.Api: Exposición de endpoints HTTP, configuración de middleware y Swagger.
- Producto.Application: Casos de uso, validaciones, DTOs y lógica de negocio.
- Producto.Domain: Entidades, interfaces y reglas de negocio puras.
- Producto.Infrastructure: Implementación de repositorios, EF Core, configuración de base de datos.
- Producto.UnitTests: Pruebas unitarias con mocks y validaciones estructurales.
Principios aplicados:
- SOLID y Dependency Injection
- Mapeo de errores con códigos HTTP y convenciones claras

Consumo de login:
El endpoint de login se encuentra en POST /api/auth/login.
Debe enviarse un JSON con los campos username y password.
La respuesta incluye un token JWT que debe usarse en los headers de las solicitudes protegidas.
Ejemplo de header:
Authorization: Bearer {token}

Uso del token en ProductsController:
El controlador de productos está protegido con el atributo [Authorize], lo que requiere un token válido para acceder a sus métodos.
Todos los endpoints como GET, POST y GET por ID requieren que el cliente incluya el token en el header Authorization.
Si no se incluye, se retorna un error 401 Unauthorized.

SQL--
CREATE DATABASE TestProductos

USE TestProductos;
go

CREATE TABLE Usuarios (
    Id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    Username NVARCHAR(100) NOT NULL UNIQUE,
    Password NVARCHAR(100) NOT NULL,
    Rol NVARCHAR(50) NOT NULL
);

INSERT INTO Usuarios (Username, Password, Rol)
VALUES ('admin', '123', 'Admin'),
       ('user1', 'abc', 'User');

CREATE TABLE Productos (
    Id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    Nombre NVARCHAR(100) NOT NULL,
    Descripcion NVARCHAR(255),
    Precio DECIMAL(18,2) NOT NULL CHECK (Precio > 0),
    Stock INT NOT NULL CHECK (Stock >= 0),
	Estado BIT NOT NULL DEFAULT 1
);

CREATE TABLE ProductoHistorial (
    Id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    ProductoId UNIQUEIDENTIFIER NOT NULL,
    FechaCambio DATETIME NOT NULL DEFAULT GETDATE(),
    CampoModificado NVARCHAR(100) NOT NULL,
    ValorAnterior NVARCHAR(255),
    ValorNuevo NVARCHAR(255),
    UsuarioModificador NVARCHAR(100) NOT NULL
);

ALTER TABLE ProductoHistorial
ADD CONSTRAINT FK_ProductoHistorial_Producto
FOREIGN KEY (ProductoId) REFERENCES Productos(Id);
