## Proyecto de Conexión Remota con PowerShell y NodeJS
Consiste en crear un Login, en el que solo podrá "iniciar sesión" el Administrador con su respectiva contraseña. 
Una vez dentro, se abre una página diferente, la cual da 3 opciones con botones, con los cuales el administrador podrá obtener información, al instante, sobre:
<PROCESOS>, <SERVICIOS> O <LA RED>. 

# Parte1: HTML 
<!DOCTYPE html>
<html>
<head>
  <title>Formulario</title>
  <link rel="stylesheet" href="estilo.css">
</head>
<body>
	 <header>
        <h1>Login Administrador</h1>
    </header>
   <main>
	<form id="login_form" class="form_class" action="mostraroperaciones" method="post">
		<div class="form_div">
			  <label>Login:</label>
			  <input class="field_class" type="text" name="login" autofocus>
			  <label>Password:</label>
			  <input class="field_class" type="password" name="password"><br>
			  <input class="submit_class" type="submit" value="acceder">
		</div>
		<div class="info_div">
                <p>Solo podrá acceder el usuario administrador.</p>
        </div>		
	</form>
   </main>
<script type="text/javascript" src="app.js"></script>
</body>
</html>

# Parte2: JavaScript 

const express = require('express')
const app = express()
const bodyParser = require('body-parser')
const path = require('path')
const { exec } = require('child_process')
app.use(express.static(__dirname + '/public'))
app.use(bodyParser.urlencoded({ extended: false }))
app.post('/mostraroperaciones', (req, res) => {
let pass = req.body.password
let name = req.body.login
	if (pass=='admin' && name=='admin'){ 		
	  let pagina = '<!doctype html><html><head><link rel="stylesheet" href="estilo.css"> </head><body>'
	  pagina +='<header><h1> BIENVENIDO ADMINISTRADOR</h1></header>'
	  pagina +='<form name="elegir" action="mostrarcosas" method="post">'
	  pagina +='<div class="div_2">'
	  pagina +='<label> Procesos </label>' 
	  pagina +='<input class="input_class" type="submit" name="primero">' 
	  pagina +='<label> Servicios </label>' 
	  pagina +='<input class="input_class" type="submit" name="segundo">' 
	  pagina +='<label> Redes </label>' 
	  pagina +='<input class="input_class" type="submit" name="tercero">'
	  pagina +='</form>'
	  pagina +='</div>'
	  pagina +='</body></html>'
	  res.send(pagina)
    } 
    else{ 
        res.sendFile(path.resolve(__dirname, 'public/index.html'));          
    } 
})

app.post('/mostrarcosas', (req, res) => { 
 let accion1 = req.body.primero
 let accion2 = req.body.segundo
 let accion3 = req.body.tercero
  if (accion1) {
		exec('Get-Process | select id, name, cpu  | sort cpu -Descending | ConvertTo-Html -CssUri "estilo2.css"', {'shell':'powershell.exe'}, (error, stdout, stderr)=> 
		{res.send(stdout);})
	}else if (accion2) {
		exec('Get-WmiObject -Class Win32_Service | Where-Object State -EQ "Running" | select ProcessId, Name | ConvertTo-Html -CssUri "estilo2.css"', {'shell':'powershell.exe'}, (error, stdout, stderr)=> 
		{res.send(stdout);})			
	}else if (accion3) {
		exec('Get-NetTCPConnection –State Established | select LocalAddress, LocalPort, RemoteAddress, RemotePort, OwningProcess | ConvertTo-Html -CssUri "estilo2.css"', {'shell':'powershell.exe'}, (error, stdout, stderr)=> 
		{res.send(stdout);})			
	}
})
var server = app.listen(8888, () => {
  console.log('Servidor web iniciado')
})

##################################################################################################################################################################
Las páginas con los estilos contienen:
# ESTILOS1
* {
    padding: 0px;
    margin: 0px;
}
body {
    background-color: lightgreen;
}
header {
    background-color: black;
    color: white;
    display: flex;
    align-items: center;
    justify-content: center;
    height: 15vh;
    box-shadow: 5px 5px 10px rgb(0,0,0,0.3);
}
h1 {
    letter-spacing: 1.5vw;
    font-family: 'system-ui';
    text-transform: uppercase;
    text-align: center;
}
main {
    display: flex;
    align-items: center;
    justify-content: center;
    height: 75vh;
    width: 100%;
    background: url(https://upload.wikimedia.org/wikipedia/commons/thumb/0/0b/Mountains-1412683.svg/1280px-Mountains-1412683.svg.png) no-repeat center center;
    background-size: cover;
}
.form_class {
    width: 500px;
    padding: 40px;
    border-radius: 8px;
    background-color: white;
    font-family: 'system-ui';
    box-shadow: 5px 5px 10px rgb(0,0,0,0.3);
}
.form_div {
    text-transform: uppercase;
}
.form_div > label {
    letter-spacing: 3px;
    font-size: 1rem;
}
.info_div {
    text-align: center;
    margin-top: 20px;
}
.info_div {
    letter-spacing: 1px;
}
.field_class {
    width: 100%;
    border-radius: 6px;
    border-style: solid;
    border-width: 1px;
    padding: 5px 0px;
    text-indent: 6px;
    margin-top: 10px;
    margin-bottom: 20px;
    font-family: 'system-ui';
    font-size: 0.9rem;
    letter-spacing: 2px;
}
.submit_class {
    border-style: none;
    border-radius: 5px;
    background-color: #FFE6D4;
    padding: 8px 20px;
    font-family: 'system-ui';
    text-transform: uppercase;
    letter-spacing: .8px;
    display: block;
    margin: auto;
    margin-top: 10px;
    box-shadow: 2px 2px 5px rgb(0,0,0,0.2);
    cursor: pointer;
}

.div_2{
	background-color: white;
	text-align: center;
}

.input_class{
	width: 100px;
    border-radius: 6px;
    border-style: solid;
    border-width: 1px;
    padding: 5px 0px;
    text-indent: 6px;
    margin-top: 10px;
    margin-bottom: 20px;
    font-family: 'system-ui';
    font-size: 0.9rem;
    letter-spacing: 2px;
}

# ESTILOS2
body { background-color:#E5E4E2;
       font-family:Monospace;
       font-size:10pt; }
td, th { border:0px solid black;
         border-collapse:collapse;
         white-space:pre; }
th { color:white;
     background-color:black; }
table, tr, td, th { padding: 0px; margin: 0px ;white-space:pre; }
table { margin-left:25px; }
h2 {
 font-family:Tahoma;
 color:#6D7B8D;
}
.footer
{ color:green;
  margin-left:25px;
  font-family:Tahoma;
  font-size:8pt;
}


