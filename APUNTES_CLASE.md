# AÑADIR USUARIOS DE UNA LISTA DE USUARIOS 
foreach($user in gc ./usuarios)
{
 bash -c "sudo useradd $user"
}
bash -c "sudo useradd pepa"

Para que al iniciar el ubuntu me salga el root directamente
ubuntu2004 config --default-user root

# CONVERTIR A JSON
$ficherojson = '[
      {
	"dia": "1",
        "id": "1",
        "tarea": "abrir"
      },
      {
	"dia": "2",
        "id": "2",
        "tarea": "cerrar"
      }
]'
$json = ConvertFrom-Json $ficherojson
$json.tarea
