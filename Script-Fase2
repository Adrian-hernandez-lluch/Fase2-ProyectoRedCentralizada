Clear-Host
Write-Host "==========================================" -ForegroundColor Cyan
Write-Host "     SCRIPT GESTIÓN ACTIVE DIRECTORY      " -ForegroundColor Cyan
Write-Host "==========================================" -ForegroundColor Cyan

# Rutas de los CSV
$csvUnidades = ".\unidades_org.csv"
$csvGrupos   = ".\grupos.csv"
$csvEquipos  = ".\equipos.csv"
$csvUsuarios = ".\usuarios.csv"

# -------------------------------
# FUNCIÓN: Crear Unidades Organizativas
# -------------------------------
function Crear-Unidades {
    Write-Host "`n[+] Creando Unidades Organizativas..." -ForegroundColor Yellow

    Import-Csv $csvUnidades -Delimiter ":" | ForEach-Object {
        $Name = $_.Name
        $Path = $_.Path

        if (-not (Get-ADOrganizationalUnit -LDAPFilter "(name=$Name)" -SearchBase $Path -ErrorAction SilentlyContinue)) {
            New-ADOrganizationalUnit -Name $Name -Path $Path
            Write-Host "   - OU creada: $Name"
        } else {
            Write-Host "   - OU ya existe: $Name"
        }
    }
}

# -------------------------------
# FUNCIÓN: Crear Grupos
# -------------------------------
function Crear-Grupos {
    Write-Host "`n[+] Creando Grupos..." -ForegroundColor Yellow

    Import-Csv $csvGrupos -Delimiter ":" | ForEach-Object {
        $GroupName = $_.Name
        $Path      = $_.Path

        if (-not (Get-ADGroup -Filter "Name -eq '$GroupName'" -ErrorAction SilentlyContinue)) {
            New-ADGroup -Name $GroupName -GroupScope Global -GroupCategory Security -Path $Path
            Write-Host "   - Grupo creado: $GroupName"
        } else {
            Write-Host "   - Grupo ya existe: $GroupName"
        }
    }
}

# -------------------------------
# FUNCIÓN: Crear Equipos
# -------------------------------
function Crear-Equipos {
    Write-Host "`n[+] Creando Equipos..." -ForegroundColor Yellow

    Import-Csv $csvEquipos -Delimiter ":" | ForEach-Object {
        $Equipo = $_.Name
        $Path   = $_.Path

        if (-not (Get-ADComputer -Filter "Name -eq '$Equipo'" -ErrorAction SilentlyContinue)) {
            New-ADComputer -Name $Equipo -Path $Path -Enabled $true
            Write-Host "   - Equipo creado: $Equipo"
        } else {
            Write-Host "   - Equipo ya existe: $Equipo"
        }
    }
}

# -------------------------------
# FUNCIÓN: Crear Usuarios
# -------------------------------
function Crear-Usuarios {
    Write-Host "`n[+] Creando Usuarios..." -ForegroundColor Yellow

    Import-Csv $csvUsuarios -Delimiter "*" | ForEach-Object {
        $Account = $_.Account

        if (-not (Get-ADUser -Filter "SamAccountName -eq '$Account'" -ErrorAction SilentlyContinue)) {

            New-ADUser `
                -Name "$($_.Name) $($_.Surname1) $($_.Surname2)" `
                -GivenName $_.Name `
                -Surname $_.Surname `
                -SamAccountName $Account `
                -UserPrincipalName "$Account@iesjaumei.mylocal" `
                -Path $_.Path `
                -EmailAddress $_.Email `
                -AccountPassword (ConvertTo-SecureString $_.Password -AsPlainText -Force) `
                -Enabled $true

            Write-Host "   - Usuario creado: $Account"

        } else {
            Write-Host "   - Usuario ya existe: $Account"
        }
    }
}

# -------------------------------
# FUNCIÓN: Asignar Usuarios a Grupos
# -------------------------------
function Asignar-Grupos {
    Write-Host "`n[+] Asignando Usuarios a Grupos..." -ForegroundColor Yellow

    Import-Csv $csvUsuarios -Delimiter "*" | ForEach-Object {
        Add-ADGroupMember -Identity $_.Group -Members $_.Account -ErrorAction SilentlyContinue
        Write-Host "   - $_.Account añadido a $_.Group"
    }
}

# -------------------------------
# FUNCIÓN: Asignar Equipos a Usuarios
# -------------------------------
function Asignar-Equipos {
    Write-Host "`n[+] Asignando Equipos a Usuarios..." -ForegroundColor Yellow

    Import-Csv $csvUsuarios -Delimiter "*" | ForEach-Object {
        $Equipo = $_.Computer
        $Usuario = $_.Account

        if ($Equipo -and $Usuario) {
            Write-Host "   - Equipo $Equipo asignado a $Usuario"
        }
    }
}

# -------------------------------
# MENÚ PRINCIPAL
# -------------------------------
do {
    Write-Host "`n========= MENÚ PRINCIPAL =========" -ForegroundColor Cyan
    Write-Host "1. Crear Unidades Organizativas"
    Write-Host "2. Crear Grupos"
    Write-Host "3. Crear Equipos"
    Write-Host "4. Crear Usuarios"
    Write-Host "5. Asignar Usuarios a Grupos"
    Write-Host "6. Asignar Equipos a Usuarios"
    Write-Host "7. Ejecutar TODO"
    Write-Host "0. Salir"
    $op = Read-Host "Selecciona una opción"

    switch ($op) {
        1 { Crear-Unidades }
        2 { Crear-Grupos }
        3 { Crear-Equipos }
        4 { Crear-Usuarios }
        5 { Asignar-Grupos }
        6 { Asignar-Equipos }
        7 {
            Crear-Unidades
            Crear-Grupos
            Crear-Equipos
            Crear-Usuarios
            Asignar-Grupos
            Asignar-Equipos
        }
        0 { Write-Host "Saliendo..." -ForegroundColor Red }
        default { Write-Host "Opción no válida" -ForegroundColor Red }
    }

} while ($op -ne 0)
