# Patrones Avanzados en React JS

[<img src="./public/cover.png">](https://www.udemy.com/course/react-js-patrones/?referralCode=FF3F91AFC79C5837D13E)



## Pattern: Custom Hook

Es simplemente es una función de JavaScript, cuyo nombre comienza con la palabra  "use", Un custom hook puede tranquilamente llamar a otros hooks.

El propósito de crear un poco más que nada es la manera de abstraer lógica de estado de un componente por lo menos.

**Ventajas**

- Favorecen el mantenimiento al tener lógica de estado separada en funciones.
- Mas fácil de probar al ser solo una función, desacoplando el código.
- Complementa otros patrones
- Evita duplicar código

**Desventajas**

- Solo se pueden usar a partir de la versión 6.8



**En que casos aplica usar este patrón?**

Básicamente cualquier componente que tenga lógica de estado es candidato a usar un custom hook, pero se justifica cuando se quiere hacer reutlización de lógica entre varios componentes.

**Ejemplos de uso**

- Formularios
- Suscripciones
- Temporizadores
- Animaciones

Ejemplos de aplicación

Tenemos la lógica de estado de consumo de una API, dentro del componente. 

El problema es que en caso de necesitar consumir la api en otro componente, no sería lo óptimo duplicar todo el código.

```jsx
import React, {useState, useEffect} from 'react';

const apiBaseUrl = 'https://api.github.com';

const url = `${apiBaseUrl}/orgs/Developero-oficial/repos?sort=created`;

export const NormalDevelopero = () => {
  const [data, setData] = useState(null);
  const [isFetching, setIsFetching] = useState(false);
  const [error, setError] = useState(null);

  useEffect(() => {
    const doFetch = async () => {
      const fetchData = async () => {
        try {
          setIsFetching(true);
          const response = await fetch(url);
          const data = await response.json();
          setData(data);
        } catch (e) {
          setError(e);
        } finally {
          setIsFetching(false);
        }
      };
      fetchData();
    };

    doFetch();
  }, []);

  if (isFetching) {
    return 'Loading...';
  }

  if (error) {
    return <p>{error.message}</p>;
  }

  if (!data) {
    return null;
  }

  return data.map(({name, html_url}) => (
    <div key={html_url}>
      <p>
        <a href={html_url} target="_blank" rel="noopener noreferrer">
          {name}
        </a>
      </p>
    </div>
  ));
};
```

La idea es traspasar la lógica de estado a un custom hook, que podemos nombrar  useFetch.js

```js
const apiBaseUrl = 'https://api.github.com';

const url = `${apiBaseUrl}/orgs/facebook/repos?sort=created`;

const useFetch = url => {
  const [data, setData] = useState(null);
  const [isFetching, setIsFetching] = useState(false);
  const [error, setError] = useState(null);

  useEffect(() => {
    const doFetch = async () => {
      const fetchData = async () => {
        try {
          setIsFetching(true);
          const response = await fetch(url);
          const data = await response.json();
          setData(data);
        } catch (e) {
          setError(e);
        } finally {
          setIsFetching(false);
        }
      };
      fetchData();
    };

    doFetch();
  }, [url]);

  return {
    data,
    isFetching,
    error,
  };
};
```

Luego podemos utilizar el custom hook en aquellos componentes que necesiten implementar esta lógica

```js
import {useFetch} from './useFetch.js'

const {data, isFetching, error} = useFetch(url);
```

**Links de interés**

https://github.com/rehooks/awesome-react-hooks

https://github.com/streamich/react-use



## Pattern: High ordered components [HOC]

