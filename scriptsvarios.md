# SCRIPTS DE POWERSHELL VARIADOS Y ÚTILES
-------------------------------------------------
## En el siguiente script buscamos conseguir unir el contenido de varios documentos de word en 1 sólo
-------------------------------------------------
```Powershell
# Lista de documentos que quieres combinar
$documentPaths = @(
    'C:\ruta\documento1.docx',
    'C:\ruta\documento2.docx'
    #agregar rutas de los docus que quiero unir
)

# Ruta del documento de salida
$resultDocument = $resultDocument = 'C:\ruta\todojunto.doc' #En mi caso he puesto la extensión .doc por que con la de .docx no me funcionaba

$word = New-Object -ComObject Word.Application
$word.Visible = $false

# Crear un nuevo documento en blanco que se utilizará para combinar los documentos
$mergedDocument = $word.Documents.Add()

# Recorrer la lista de documentos y copiar su contenido al documento combinado
foreach ($documentPath in $documentPaths) {
    $range = $mergedDocument.Range()
    
    # Mover al final del documento combinado
    $range.Collapse(0)  # wdCollapseEnd
    
    # Insertar un salto de página para que el siguiente documento comience en una nueva página
    $range.InsertBreak(7)  # wdPageBreak
    
    # Insertar el contenido del documento actual al documento combinado
    $range.InsertFile($documentPath)
}

# Guardar el documento combinado
$mergedDocument.SaveAs($resultDocument)

# Cerrar Word y liberar recursos
$word.Quit()
$null = [System.Runtime.Interopservices.Marshal]::ReleaseComObject($range)
$null = [System.Runtime.Interopservices.Marshal]::ReleaseComObject($mergedDocument)
$null = [System.Runtime.Interopservices.Marshal]::ReleaseComObject($word)
[System.GC]::Collect()
[System.GC]::WaitForPendingFinalizers()

```
