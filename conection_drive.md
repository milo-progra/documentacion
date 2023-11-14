## contectar con drive mediane auth2
*   ir acredenciales
*   crear credenciales
*   crear credenciales oauth2 para aplicacion de escritorio
*   descargar credenciales y cambiar nombre credentials.json y guardarlo en la raiz

## crear token.json para tener acceso a los servicios de google
*   Si es la primera vez que utilizas el sistema se creara un token par tener acceso a los servicios de drive
``` python

    if os.path.exists("token.json"):
            creds = Credentials.from_authorized_user_file("token.json", SCOPES)
        # If there are no (valid) credentials available, let the user log in.
        if not creds or not creds.valid:
            if creds and creds.expired and creds.refresh_token:
                creds.refresh(Request())
            else:
                flow = InstalledAppFlow.from_client_secrets_file(
                    "credentials.json", SCOPES
                )
            creds = flow.run_local_server(port=0)
            # Save the credentials for the next run
            with open("token.json", "w") as token:
                token.write(creds.to_json())
```

## Listar archivos de drive

``` python
        service = build("drive", "v3", credentials=creds)

        # Call the Drive v3 API
        results = (
            service.files()
            .list(pageSize=10, fields="nextPageToken, files(id, name)")
            .execute()
        )
        items = results.get("files", [])

        if not items:
            print("No files found.")
            print("Files:")
            return
        for item in items:
            print(f"{item['name']} ({item['id']})")    
```

## Subir un archivo en drive

*   Subir archivo a drive
``` python

    file_metadata = {
        "name": "prueba_planilla_python",
        "mimeType": "application/vnd.google-apps.spreadsheet"
    }
    nombre = "Fabian Salazar sepulveda"
    media = MediaFileUpload("test.json", mimetype="application/vnd.openxmlformats-officedocument.spreadsheetml.sheet")
    file = service.files().create(body=file_metadata, media_body=media, fields="id").execute()

    print(f'File ID: {file.get("id")}')
```
## Codigo completo en una cloud function

``` python
    @https_fn.on_request()
    def init_drive(req: https_fn.Request) -> https_fn.Response: 
        """
            Crear archivo de token.json utilizando credenciales de auth2
        """
        SCOPES = ["https://www.googleapis.com/auth/drive.file", "https://www.googleapis.com/auth/drive"]
        creds = None
        
        if os.path.exists("token.json"):
            creds = Credentials.from_authorized_user_file("token.json", SCOPES)
        # If there are no (valid) credentials available, let the user log in.
        if not creds or not creds.valid:
            if creds and creds.expired and creds.refresh_token:
                creds.refresh(Request())
            else:
                flow = InstalledAppFlow.from_client_secrets_file(
                    "credentials.json", SCOPES
                )
            creds = flow.run_local_server(port=0)
            # Save the credentials for the next run
            with open("token.json", "w") as token:
                token.write(creds.to_json())

        try:
            service = build("drive", "v3", credentials=creds)

            """ listar todos los archivos de carpeta raiz de drive """
            # Call the Drive v3 API
            results = (
                service.files()
                .list(pageSize=10, fields="nextPageToken, files(id, name)")
                .execute()
            )
            items = results.get("files", [])

            if not items:
                print("No files found.")
                print("Files:")
                return
            for item in items:
                print(f"{item['name']} ({item['id']})")

            """ Subir archivos de carpeta raiz de drive """
            #Subir archivo a drive
            file_metadata = {
                "name": "prueba_planilla_python",
                "mimeType": "application/vnd.google-apps.spreadsheet"
            }
            nombre = "Fabian Salazar sepulveda"
            media = MediaFileUpload("test.json", mimetype="application/vnd.openxmlformats-officedocument.spreadsheetml.sheet")
            file = service.files().create(body=file_metadata, media_body=media, fields="id").execute()

            print(f'File ID: {file.get("id")}')

            return https_fn.Response("test drive")
        except HttpError as error:
            # TODO(developer) - Handle errors from drive API.
            print(f"An error occurred: {error}")

```

