# Cambiar el secreto de una aplicaci�n

**Aspen** permite al propietario de una aplicaci�n, cambiar o actualizar el secreto para firmar las solicitudes.

Cambiar el secreto de su aplicaci�n al igual que en la [solicitud de un token de autenticaci�n](JWT-Request.md) necesitar� de las cabeceras de autenticaci�n personalizadas junto con un par�metro que recibe el valor para el nuevo secreto.

Tendr� que invocar a la operaci�n `Secret` del servicio de autenticaci�n, a trav�s de una operaci�n `POST` agregando las dos [cabeceras personalizadas](JWT-Request.md#cabeceras-de-autenticacion-requeridas) usando su `AppKey` y `AppSecret` actual.

### Implementaci�n demostrativa

Usando el [c�digo de demostraci�n](samples/docs/Demo.cs) agregaremos a la interfaz `IAspenService` una nueva funci�n con el nombre `UpdateSecret`.

```c#
public interface IAspenService
{
    AuthContext AuthContext { get; }
    void Signin();
    IList<DocType> GetDocTypes();
    void UpdateSecret(string newSecret);
}
```

Y en la clase `AspenService` implementaremos la nueva funci�n:

```c#
public void UpdateSecret(string newSecret)
{
    var payload = new Dictionary<string, object>
    {
        { "Nonce", this.nonceProvider.GetNonce() },
        { "Epoch", this.epochProvider.GetSeconds() }
    };

    IRestRequest request = new RestRequest("/app/auth/secret", Method.POST);
    request.AddHeader(AppHeaderKey, this.appKey);
    request.AddHeader(PayloadHeaderKey, this.encoder.Encode(payload, this.appSecret));
    request.AddParameter("NewValue", newSecret);
    IRestResponse response = this.restClient.Execute(request);
    if (response.IsSuccessful)
    {
        return;
    }

    throw new AspenException(response);
}
```

Una vez que el cambio de secreto haya finalizado con �xito, toda nueva solicitud firmada usando las credenciales anteriores, autom�ticamente ser� rechazada.
 
### Caracter�sticas de un secreto fuerte
El secreto debe cumplir con un alto nivel de complejidad para que sea poco o nada predecible y mitigar ataques de fuerza bruta.

Prepare nuevos secretos para su aplicaci�n que incluyan las siguientes caracter�sticas:

* :abc: Letras min�sculas.
* :ab: Letras may�sculas.
* :1234: N�meros.
* :interrobang: Caracteres especiales.
* :arrow_right: Longitud entre 128 y 214 caracteres.

```c#
// Una forma sencilla de generar claves fuertes con .NET:
System.Web.Security.Membership.GeneratePassword(128, 0);
```

Una herramienta en l�nea para generar claves seguras: [Strong Random Password Generator](https://passwordsgenerator.net/)