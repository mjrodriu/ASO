## Esta es la primera parte en la que se crea el formulario y el fichero JSON.
using assembly System.Windows.Forms
using namespace System.Windows.Forms
class usuario
{
[String]$nombre
[string]$operacion
usuario([string]$nombre, [string]$operacion)
{
$this.nombre = $nombre
$this.operacion = $operacion
}
}
$arrayoperaciones = New-Object System.Collections.ArrayList
$form = [Form] @{
Text = 'Active Directory'
Size = New-Object System.Drawing.Size(500,500)
StartPosition = 'CenterScreen'
}
$label = [Label]@{
Text = 'nombre:'
Location = New-Object System.Drawing.Point(100,40)
Size = New-Object System.Drawing.Size(280,20)
}
$label2 = [Label]@{
Text = 'operacion:'
Location = New-Object System.Drawing.Point(100,90)
Size = New-Object System.Drawing.Size(280,20)
}
$TextBox = [TextBox] @{
Location = New-Object System.Drawing.Size(100,60)
Size = New-Object System.Drawing.Size(260,20)
}
$TextBox2 = [TextBox] @{
Location = New-Object System.Drawing.Size(100,110)
Size = New-Object System.Drawing.Size(260,20)
}

$button = [Button] @{
Text = 'OK'
Location = New-Object System.Drawing.Point(130,200)
Size = New-Object System.Drawing.Size(75,23)
}
$buttoncancel = [Button] @{
Text = 'Cancelar'
Location = New-Object System.Drawing.Size(230,200)
Size = New-Object System.Drawing.Size(75,23)
}
$button.add_Click{
Write-Host $TextBox.Text
Write-Host $TextBox2.Text
$arrayoperaciones.add([usuario]::new($TextBox.Text, $TextBox2.Text))
}
$form.Controls.Add($label)
$form.Controls.Add($label2)
$form.Controls.Add($TextBox)
$form.Controls.Add($textBox2)
$form.Controls.Add($button)
$form.Controls.Add($buttoncancel)
$form.ShowDialog()

$arrayoperaciones | ConvertTo-Json | Out-File C:\xampp\htdocs\usuarios.txt -Append -Encoding default

## En la segunda parte, se lee el fichero JSON y hace las operaciones introducidas en el formulario.
$invocar = Invoke-RestMethod "http://localhost/usuarios.txt" 
foreach($linea in $invocar)
{
    if($linea.operacion -eq "crear")
    {
        $password = ConvertTo-SecureString -string "Andel1928" -AsPlainText -Force
        New-ADUser -name ($linea.nombre) -Path ("OU=asir1,dc=andel,dc=local") -AccountPassword  $password -Enable $true 
    }
    elseif($linea.operacion -eq "unir")
    {
        Add-ADGroupmember -Members $linea.nombre -Identity alumnos 

    }
    elseif($linea.operacion -eq “eliminar")
    {
    Remove-ADUser -Identity $linea.nombre -Confirm:$False        
    }
}
