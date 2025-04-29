Para crear en ***Environments*** (Archivo de variables) ejecutamos el siguiente comando

> ng generate environments

nota: si esta corriendo la aplicasion hay que detenerla ( ***Ctrol + c*** ) y volverla a correr ( ***ng serve*** )

Se crean dos archivos con los siguientes nombre:

- environment.development.ts  --- (desarrollo)
- environment.ts ------------------ (produccion)

***environment.development.ts***

```ts
export const environment = {
  apiUrl: 'https://localhost:7012',
  apiUrlWear: 'https://localhost:7012/weatherforecast'
}
```

***environment.ts***
```ts
export const environment = {
  apiUrl: 'PENDING'
}
```