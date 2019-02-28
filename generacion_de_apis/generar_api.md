# Generar API Beego

En está sección se realiza  paso a paso la creación de una API con el framework Beego


## Requerimientos

1. [Instalación Golang](/instalacion_de_herramientas/golang.md)
3. [Instalación Beego y Bee](/instalacion_de_herramientas/beego.md)
4. [Instalación Postgres](/instalacion_de_herramientas/postgres.md)
5. [Instalación pgAdmin3](/instalacion_de_herramientas/pgadmin3.md)
6. [Instalación pgModeler](/instalacion_de_herramientas/pgmodeler.md)

## Generar API

1. Crear un bd llamada bd_oas

  ![Crear BD](/generacion_de_apis/img/001.png)

2. Crear una tabla usuario

  ![Crear Tabla](/generacion_de_apis/img/002.png)

  - Puedes ejecutar el sql [adjunto](/generacion_de_apis/bd/usuario.sql)

         psql -U postgres -d bd_oas -a -f usuario.sql

  - puedes exportar el modelo dbm desde el pgModeler

3. Crear directorio para proyecto Beego

       cd ~/go/src/github.com/ && mkdir TuUsuarioGithub

    Ingrer al directorio

       cd ~/go/src/github.com/TuUsuarioGithub

4. Crear API

       bee api testApi -driver=postgres -conn=postgres://postgres:postgres@127.0.0.1/bd_oas?sslmode=disable

    Se Creara un directorio llamado testApi con los archivo correspondiente a la api.

       ├── conf
       │   └── app.conf
       ├── controllers
       │   └── usurio.go
       ├── main.go
       ├── models
       │   └── usurio.go
       ├── routers
       │   └── router.go
       └── tests

    Especificar el esquema en el proyecto. Para esto, editamos el archivo **testApi/conf/app.conf** agregamos lo siguiente **"&search_path=nombre_de_tu_schema"**

    - Código original:

          sqlconn = postgres://postgres:postgres@127.0.0.1/bd_oas?sslmode=disable

    - Ajuste:

          sqlconn = postgres://postgres:postgres@127.0.0.1/bd_oas?sslmode=disable&search_path=public

    Especificamos el auto incremental del id en los modelos en el archivo **testApi/models/usurio.go**

    - Código original:

          type Usurio struct {
            Id     int    `orm:"column(id);pk"`
            Nombre string `orm:"column(nombre);null"`
          }

    - Ajuste:

          type Usurio struct {
            Id     int    `orm:"column(id);pk;auto"`
            Nombre string `orm:"column(nombre);null"`
          }

5. Configurar cors

  Esta configuración permitirá quee los servicios sean consumibles desde un navegador web

  - En el import()

         "github.com/astaxie/beego/plugins/cors"

  - En la Funcion func init()

         beego.InsertFilter("*", beego.BeforeRouter, cors.Allow(&cors.Options{
           AllowOrigins: []string{"*"},
           AllowMethods: []string{"PUT", "PATCH", "GET", "POST", "OPTIONS", "DELETE"},
           AllowHeaders: []string{"Origin", "x-requested-with",
             "content-type",
             "accept",
             "origin",
             "authorization",
             "x-csrftoken"},
           ExposeHeaders:    []string{"Content-Length"},
           AllowCredentials: true,
         }))

  - Al final tendremos la funcion main de la siguieten forma:

         func main() {
           orm.RegisterDataBase("default", "postgres", beego.AppConfig.String("sqlconn"))
           if beego.BConfig.RunMode == "dev" {
             beego.BConfig.WebConfig.DirectoryIndex = true
             beego.BConfig.WebConfig.StaticDir["/swagger"] = "swagger"
           }

           beego.InsertFilter("*", beego.BeforeRouter, cors.Allow(&cors.Options{
             AllowOrigins: []string{"*"},
             AllowMethods: []string{"PUT", "PATCH", "GET", "POST", "OPTIONS", "DELETE"},
             AllowHeaders: []string{"Origin", "x-requested-with",
               "content-type",
               "accept",
               "origin",
               "authorization",
               "x-csrftoken"},
             ExposeHeaders:    []string{"Content-Length"},
             AllowCredentials: true,
           }))

           beego.Run()
         }

6. Generar Documentación

  Ubicado en la rais del proyecto

       cd ~/go/src/github.com/TuUsuarioGithub/testApi

  Ejecutra

       bee run -downdoc=true -gendoc=true