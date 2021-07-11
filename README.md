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

**Ejemplos de aplicación**

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

Es una función javascript que recibe por parámetro un componente react y retorna una versión de ese componente enriquecido, inyectandole nueva lógica. Se nombre con la palabra "with" al inicio.  Se utiliza para reutilizar lógica de comportamiento.

**Ventajas**

- Favorece el mantenimiento al tener lógica separada en una función que tenga sólo una razón para cambiar, que es el principio de responsabilidad única.
- Nos ayuda a eliminar duplicidad

**Desventajas**

- Colición de nombres de props.

**En que casos aplica usar este patrón?**

Cada vez que necesitamos abstraer y reutilizar lógica de comportamiento entre componentes

**Ejemplos de aplicación**

Este es el ejemplo de un formulario, vamos a crear un HOC, que nos permita poder crear formularios independientemente de los campos que tenga adentro, y que pueda reutilizar la lógica de manejo de formulario.

normal-form.js

```jsx
import React, {useState} from 'react';
import PropTypes from 'prop-types';

export const NormalForm = ({onSubmit}) => {
  const [formValues, setFormValues] = useState({
    name: '',
    address: '',
  });

  const handleChange = e => {
    const {
      target: {name, value},
    } = e;
    setFormValues({...formValues, [name]: value});
  };

  const handleSubmit = e => {
    e.preventDefault();
    onSubmit(JSON.stringify(formValues));
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <p>Name</p>
        <input
          type="text"
          name="name"
          value={formValues.name}
          onChange={handleChange}
        />
      </div>
      <div>
        <p>Address</p>
        <input
          type="text"
          name="address"
          value={formValues.address}
          onChange={handleChange}
        />
      </div>
      <div>
        <button type="submit">Send</button>
      </div>
    </form>
  );
};

```

Creamos el hoc withControlledForm, que recibe como parámetro un componente y el estado inicial de los campos, y lo devuelve con la lógica inyectada para manejar formularios.

withControlledForm.jsx

```jsx
const withControlledForm = (Form, initialState = {}) => {
  const ControlledForm = ({onSubmit}) => {
    const [formValues, setFormValues] = useState(initialState);

    const handleChange = e => {
      const {
        target: {name, value},
      } = e;
      setFormValues({...formValues, [name]: value});
    };

    const handleSubmit = e => {
      e.preventDefault();
      onSubmit(formValues);
    };

    return (
      <Form
        formValues={formValues}
        handleChange={handleChange}
        handleSubmit={handleSubmit}
      />
    );
  };

  return ControlledForm;
};
```

Con esto podemos utilizarlo para componentes que necesiten de lógica de formulario.

Aquí exportamos un componente de formulario enriquecido por el HOC withControlledForm

```jsx
const MyFormA = ({formValues, handleChange, handleSubmit}) => {
  return (
    <form onSubmit={handleSubmit}>
      <div>
        <p>Name</p>
        <input
          type="text"
          name="name"
          value={formValues.name}
          onChange={handleChange}
        />
      </div>
      <div>
        <p>Job Title</p>
        <input
          type="text"
          name="jobTitle"
          value={formValues.jobTitle}
          onChange={handleChange}
        />
      </div>
      <button type="submit">Submit</button>
    </form>
  );
};

export const MyFormAControlled = withControlledForm(MyFormA, {
  name: 'john doe',
  jobTitle: 'full stack developer',
});
```

**Links de interés**

https://github.com/juan-carlos-correa/with-controlled-form

https://reactrouter.com/web/api/withRouter

## Pattern: Extendible Styles

Este patrón le da la capacidad a los componentes de poder extender sus estilos.

**Ventajas**

- Componentes abiertos a extensión y cerrados a su modificación (Solid)

**Desventajas**

- 

**En que casos aplica usar este patrón?**

Para evitar tener lógica de estilos dentro del componente, o evitar crear componentes similares innecesariamente con pequeñas diferencias de estilos.

**Ejemplos de aplicación**

normal-button.js

```jsx
import PropTypes from 'prop-types';

import './button.css';

export const Button = ({name, type, onClick, children}) => {
  return (
    <button
      name={name}
      type={type}
      onClick={onClick}
      className="btn btn-primary"
    >
      {children}
    </button>
  );
};
```

custom-button.js

Como vemos este componente está abierto a cambios de estilos, recibe como prop el className y también styles.

```jsx
import PropTypes from 'prop-types';

import './button.css';

export const CustomButton = ({
  name,
  type,
  onClick,
  children,
  className = 'btn btn-primary',
  style = {},
}) => {
  return (
    <button
      name={name}
      type={type}
      onClick={onClick}
      className={className}
      style={style}
    >
      {children}
    </button>
  );
};
```

**Links de interés**

https://material-ui.com/api/button/

https://github.com/couds/react-bulma-components/blob/master/src/components/button/button.js 

https://github.com/react-bootstrap/react-bootstrap/blob/master/src/Button.tsx





