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



## Pattern: Compound 

Este patrón aprovecha al máximo la composición de componentes para que podamos implementar una UI de una manera muy flexible, siempre y cuando esos componentes compartan un estado en común, internamente suele usar context para comunicar los componentes.

**Ventajas**

- Nos permite aplicar estructuras de UI flexibles

**Desventajas**

- Al usar React Context hace que su los componentes sean más limitados a usar por fuera de esa estructura.

**Ejemplos de uso**

Por ejemplo tenemos un blog, y tenemos componentes como título, sidebar, post, este patrón nos permite que en diferentes páginas estos componentes puedan aparecer en distinto orden, o sea nos da flexibildad, sin necesidad de incluir condicionales de renderizado dentro.

**En que casos aplica usar este patrón?**

Se utiliza solo cuando identifico que necesito una estructura de UI flexible y no antes, cómo si fuera un refactor.

**Ejemplos de aplicación**

Tenemos un componente Todo, que está conformado por los componentes

1. TodoTitle
2. TodoForm
3. TodoList

El primer problema es que al querer pasar props, deberían pasar todas por el componente TODO, recibirlos y pasarlos a los componentes hijos, si fueran muchos componentes hijos y estos tuvieran a su vez mas hijos se tornaría complejo.

El segundo problema es la legibilidad.

Tercero si yo quisiera cambiar el orden de aparición de los componentes, tendría que modificarlo, o crear otro componente TODO, y no estaría teniendo componentes reutilizables, o terminaría agregando condicionales, pero agregaríamos complejidad accidental.

Ejemplo  sin el patrón

```jsx
import {useState} from 'react';
import PropTypes from 'prop-types';

export const Todo = ({title}) => {
  const [listTodos, setListTodos] = useState({});

  const handleSubmit = inputValue => {
    setListTodos({
      ...listTodos,
      [inputValue]: {name: inputValue, isDone: false},
    });
  };

  const toogleTodo = ({target: {name}}) => {
    setListTodos({
      ...listTodos,
      [name]: {
        ...listTodos[name],
        isDone: !listTodos[name].isDone,
      },
    });
  };

  const getTodoValues = () => Object.values(listTodos);

  const todoListValues = getTodoValues();

  return (
    <div
      style={{
        boxShadow: '0 4px 8px 0 rgba(0, 0, 0, 0.2)',
        transition: '0.3s',
        borderRadius: '5px',
        padding: '8px',
      }}
    >
      <TodoTitle>
        <h2>{title}</h2>
      </TodoTitle>
      <main>
        <TodoForm onSubmit={handleSubmit} />
      </main>
      <footer>
        {!todoListValues.length && <p>No todo added yet.</p>}
        <TodoList list={todoListValues} onChange={toogleTodo} />
      </footer>
    </div>
  );
};
```

Ejemplo con el Patrón:

Así quedaría el TODO, pasaríamos directamente los componentes dentro, lo que nos da flexibilidad, para acomodarlos a gusto.

```jsx
    <FinalTodo>
      <TodoTitle>
        <h1>Compound Pattern :D</h1>
      </TodoTitle>
      <TodoForm />
      <TodoList />
    </FinalTodo>
```

FinalTodo.jsx

Creamos un context que servirá para poder comunicar todos los componentes hijos sin necesidad de hacer drill down de props.

```jsx
import {useState, createContext, useContext} from 'react';
import PropTypes from 'prop-types';

const TodoContext = createContext();

const {Provider} = TodoContext;

export const FinalTodo = ({children}) => {
  const [listTodos, setListTodos] = useState({});

  const handleSubmit = inputValue => {
    setListTodos({
      ...listTodos,
      [inputValue]: {name: inputValue, isDone: false},
    });
  };

  const toogleTodo = ({target: {name}}) => {
    setListTodos({
      ...listTodos,
      [name]: {
        ...listTodos[name],
        isDone: !listTodos[name].isDone,
      },
    });
  };

  const getTodoValues = () => Object.values(listTodos);

  const valuesProvider = {
    handleSubmit,
    toogleTodo,
    getTodoValues,
  };

  return (
    <div
      style={{
        boxShadow: '0 4px 8px 0 rgba(0, 0, 0, 0.2)',
        transition: '0.3s',
        borderRadius: '5px',
        padding: '8px',
      }}
    >
      <Provider value={valuesProvider}>{children}</Provider>
    </div>
  );
};
```

Ajustamos componentes hijos al uso de context con useContext

```jsx
export const TodoTitle = ({children}) => <header>{children}</header>;

export const TodoForm = () => {
  const [inputValue, setInputValue] = useState('');

  const {handleSubmit} = useContext(TodoContext);

  const handleInputChange = ({target: {value}}) => {
    setInputValue(value);
  };

  const _handleSubmit = e => {
    e.preventDefault();
    handleSubmit(inputValue);
    setInputValue('');
  };

  return (
    <form onSubmit={_handleSubmit}>
      <label>
        New todo:
        <input
          type="text"
          value={inputValue}
          onChange={handleInputChange}
          required
        />
      </label>
      <button type="submit">Add</button>
    </form>
  );
};

export const TodoList = () => {
  const {toogleTodo, getTodoValues} = useContext(TodoContext);

  const list = getTodoValues();

  return (
    <ul>
      {list.map(({name, isDone}) => (
        <li key={name}>
          <label>
            <input
              name={name}
              type="checkbox"
              checked={isDone}
              onChange={toogleTodo}
            />
            <span style={{textDecoration: isDone ? 'line-through' : ''}}>
              {name}
            </span>
          </label>
        </li>
      ))}
    </ul>
  );
};
```

**Links de interes:**

https://material-ui.com/es/components/cards/ (Componente que usa compound patter)

## Pattern: Render Props

Lo que hacemos con este patrón es utilizar un prop para delegar la responsabilidad de hacer un render con un componente. Esto nos deja el componente abierto y flexible a la lógica de cómo vamos a hacer render al componente o a quien vaya a utilizar esto. El componente padre decide que renderizar.

**Ventajas**

- El componente padre decide que lógica renderizar, por lo que hace reutilizable el componente hijo

**Desventajas**

- Cuando hay un encadenamiento de componentes que utilizan render props, podría llegarse a complejizar mucho la lógica de render

**Ejemplos de aplicación**

Tenemos un componente que renderiza mensajes de error, queremos convertirlo en un componente reutilizable para que renderice distinto contenido de error según la sección.

**Ejemplos sin patrón**

En este ejemplo el componente solo se limita a mostrar el mensaje de error de react, sin ninguna personalización

```jsx
import {Component} from 'react';

export class ErrorBoundary extends Component {
  state = {hasError: false, error: null};

  componentDidCatch(error) {
    this.setState({hasError: true, error});
  }

  render() {
    const {hasError, error} = this.state;
    const {children} = this.props;

    if (hasError) {
      return (
        <div>
          <p>Oops! ha ocurrido un error inesperado</p>
          {error.toString()}
        </div>
      );
    }

    return children;
  }
}
```

**Ejemplos con patrón**

En este caso si llega una prop render, renderizará el contenido pasado en la función render, y esta recibira el error, para luego formatearlo como se deseé.

```jsx
import {Component} from 'react';

export class FinalErrorBoundary extends Component {
  state = {hasError: false, error: null};

  componentDidCatch(error) {
    this.setState({hasError: true, error});
  }

  render() {
    const {hasError, error} = this.state;
    const {children} = this.props;

    if (hasError && !this.props.render) {
      return (
        <div>
          <p>Oops! ha ocurrido un error inesperado</p>
          {error.toString()}
        </div>
      );
    }

    if (hasError && this.props.render) {
      return this.props.render(error);
    }

    return children;
  }
}
```

Implementación desde el componente padre

Como vemos en la prop render del componente FinalErrorBoundary, devolvemos una función con el contenido a renderizar.

```jsx
import React, {useState} from 'react';
import {ErrorBoundary} from './error-boundary';
import {FinalErrorBoundary} from './final-error-boundary';

const MyBug = () => {
  const [isError, setIsError] = useState(false);

  const handleCrash = () => {
    setIsError(true);
  };

  if (isError) {
    throw new Error('nani?');
  }

  return <button onClick={handleCrash}>Crashear la app</button>;
};

export const RenderPropsPage = () => (
  <>
    <FinalErrorBoundary render={error => <p>{`Ups D: ${error.message}`}</p>}>
      <MyBug />
    </FinalErrorBoundary>
  </>
);
```

**Links de interes:**

https://medium.com/componentdidblog/use-a-render-prop-50de598f11ce (Articulo sobre beneficios del patrón)

https://es.reactjs.org/docs/render-props.html (Doc react)

https://www.npmjs.com/search?q=render%20props  (Proyectos con render props)



## Pattern: Control Props

Este patrón nos permite que cualquier tipo de componente que maneja un estado interno, pueda delegar el control o la manipulación de su estado por medio, o más bien a su componente padre o a quien sea que lo esté utilizando.

**Ventajas**

- Nos permite delegar el uso del estado, cntrol o la lógica de actualización del estado, Esto nos permite tener o darle flexibilidad al componente sin estar agregando complejidad innecesaria dentro de su implementación. Con esto aplicamos Open Closed principle dejando abierta la puerta a que podamos modificar su comportamiento.

**Desventajas**

- La desventaja es que el componente padre va a tener que controlar esas variables de estado y lógica.

**Ejemplo de aplicación**

Es aplicará en casos en los que tengamos un fuente cuyo cuyo estado interna creamos que está abierto a casos de uso o escenarios diferentes a los que fue este componente diseñado originalmente.

**Ejemplos sin patrón**

En este ejemplo tenemos un componente que nos muestra un boton de like y aplica un contador, manejando su lógica de estado interna y solo cumple con un escenario, al ser clickeado sumar 1 al contador.

```jsx
import {useState} from 'react';

export const LikeButton = ({cb}) => {
  const [likes, setLikes] = useState(0);

  const handleClick = () => setLikes(cb(likes));

  return (
    <button onClick={handleClick}>
      <span role="img" aria-label="like">
        👍
      </span>
      {likes}
    </button>
  );
};
```

**Ejemplos con patrón**

Con este ejemplo, el componente final, queda abierto a poder ser controlada su actualización desde afuera, e incluso no solo dejarlo fijo a manejar el evento click.

```jsx
import {useState} from 'react';

export const FinalLikeButton = ({value, setValue}) => {
  const isControlled = value !== undefined && setValue !== undefined;

  const [likes, setLikes] = useState(0);

  const handleClick = () => (isControlled ? setValue() : setLikes(likes + 1));

  return (
    <button onClick={handleClick}>
      <span role="img" aria-label="like">
        👍
      </span>
      {isControlled ? value : likes}
    </button>
  );
};
```

Componente padre que controla la lógica, implementando el manejo de estados.

```jsx
import React, {useState} from 'react';

import {LikeButton} from './like-button';
import {FinalLikeButton} from './final-like-button';

export const ControlPropsPage = () => {
  const [counter, setCounter] = useState(0);

  const handleUpdateCounter = () => {
    setCounter(counter + 5);
  };

  const handleChange = ({target: {value}}) => {
    if (value === 'like') {
      setCounter(counter + 1);
    }
  };

  return (
    <>
      <h2>Ejemplo sin Control Props</h2>
      <LikeButton cb={likes => likes + 100} />
      <hr />

      <h2>Ejemplo con Control Props</h2>
      <input type="text" onChange={handleChange} />
      <FinalLikeButton value={counter} setValue={handleUpdateCounter} />
    </>
  );
};
```

**Links de interes:**

https://github.com/downshift-js/downshift#control-props

## Pattern: Props Getters

Este patrón consiste en pasar una función como prop desde un componente padre a un componente hijo, de esta forma el componente hijo tiene acceso a los valores del componente padre.

**Ventajas**

- Nos permite el acceso a las props o valores internos de un componente o un custom hook dando posibilidad de extenderlos o modificarlos
- Nos da flexibilidad de como queremos personalizar o mostrar los componentes

**Desventajas**

- Para implementarlo debemos hacerlo en conjunto de otros patrones, puede ser customHook o render props, por lo cual es un patron complementario.

**En que casos aplica usar este patrón?**

Básicamente hay que aplicarlo cuando quieres proveer un acceso a los props de tu componente de valores internos, o tenga un componente o de un custom hook tiene una manera centralizada o cuando quieres hacer la mismo. Pero esos valores internos para ser extendidos.

**Ejemplos sin patrón**

En este caso tenemos un componente Hijo que tiene la lógica para implementar formularios, se hace uso del patrón renderProps, pasando la prop children en forma de función ( entregando al hijo lo que deba renderizar ).

El problema yace cuando queremos extender el comportamiento del componente hijo, por ejemplo que el handleChange haga algo extra.

```jsx
import {FormWithRenderProps} from './form-with-render-props';

export const PropsGettersPage = () => {
  const onSubmit = values => alert(JSON.stringify(values));

  const _handleChange = handleChange => e => {
    alert('change');
    handleChange(e);
  };

  return (
    <>
      <h2>Ejemplo sin Props Getters</h2>
      <FormWithRenderProps initialState={{name: '', jobTitle: ''}}>
        {({formValues, handleChange, handleSubmit}) => (
          <form onSubmit={handleSubmit(onSubmit)}>
            <div>
              <p>Name</p>
              <input
                type="text"
                name="name"
                value={formValues.name}
                onChange={_handleChange(handleChange)}
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
            <button style={{margin: '8px 0'}} type="submit">
              Submit
            </button>
          </form>
        )}
      </FormWithRenderProps>

      <hr />

      <h2>Ejemplo con Props Getters</h2>
    </>
  );
};
```

Componente hijo (renderiza el children), tampoco serviría el caso de modificar las funciones internas ya que se modificarían en todas las implementaciones de este componente

```jsx
export const FormWithRenderProps = ({initialState, children}) => {
  const [formValues, setFormValues] = useState({...initialState});

  const handleChange = ({target}) => {
    const {name, value} = target;

    setFormValues({...formValues, [name]: value});
  };

  const handleSubmit = _handleSubmit => event => {
    event.preventDefault();

    _handleSubmit(formValues);
  };

  const getStateAndHelpers = () => ({
    formValues,
    handleChange,
    handleSubmit,
  });

  return children(getStateAndHelpers());
};
```

**Ejemplos con patrón**

**Uso de Props Getter con Render Props** 

Abrimos la opción de que la implementación de FinalFormWIthRenderProps sea altamente extensible.                                    

Creamos una función interna que se encargará de recibir las implementaciónes que van a extender el comportamiento, como vemos a continuación, para el prop onChange llamamos a una función que llamará la implementación propia del componente y luego la obtenida por props.

```
  const getInputProps = (props = {}) => ({
    onChange: callAll(props.onChange, handleChange),
  });
```

Así quedaría finalmente el componente, con una función callAll, que se encargará de ejecutar todas las implementaciones proporcionadas.

```jsx
function callAll(...fns) {
  return function (...args) {
    fns.forEach(fn => fn && fn(...args));
  };
}

export const FinalFormWithRenderProps = ({initialState, children}) => {
  const [formValues, setFormValues] = useState({...initialState});

  const handleChange = ({target}) => {
    const {name, value} = target;

    setFormValues({...formValues, [name]: value});
  };

  const handleSubmit = _handleSubmit => event => {
    event.preventDefault();

    _handleSubmit(formValues);
  };

  const getInputProps = (props = {}) => ({
    onChange: callAll(props.onChange, handleChange),
  });

  const getStateAndHelpers = () => ({
    formValues,
    handleSubmit,
    getInputProps,
  });

  return children(getStateAndHelpers());
};

```

Así quedaría la implementación, se hace un spread de getInputProps y se pasa la implementación que se quiere utilizar , en este caso logChange.

```jsx
import React from 'react';

import {FormWithRenderProps} from './form-with-render-props';
import {FinalFormWithRenderProps} from './final-with-render-props';
import {FormWithHoc} from './form-with-hoc';
import {FormWithHook} from './form-with-hook';

export const PropsGettersPage = () => {
  const onSubmit = values => alert(JSON.stringify(values));
    
  const logChange = () => {
    console.log('changed!');
  };

  return (
    <>
      <h2>Ejemplo con Props Getters y Render Props</h2>
      <FinalFormWithRenderProps initialState={{name: '', jobTitle: ''}}>
        {({formValues, getInputProps, handleSubmit}) => (
          <form onSubmit={handleSubmit(onSubmit)}>
            <div>
              <p>Name</p>
              <input
                type="text"
                name="name"
                value={formValues.name}
                {...getInputProps({onChange: logChange})}
              />
            </div>
            <div>
              <p>Job Title</p>
              <input
                type="text"
                name="jobTitle"
                value={formValues.jobTitle}
                {...getInputProps({onChange: logChange})}
              />
            </div>
            <button style={{margin: '8px 0'}} type="submit">
              Submit
            </button>
          </form>
        )}
      </FinalFormWithRenderProps>
   </>
  );
};
```

**Uso de Props Getter con HOC**

En este caso implementamos el form con un HOC, reutilizamos la función callAll, para poder ejecutar las funciones que extienden nuestro componente.

```jsx
function callAll(...fns) {
  return function (...args) {
    fns.forEach(fn => fn && fn(...args));
  };
}

export const withControlledForm = (FormComponent, initialState = {}) =>
  class WithFormMethodsHOC extends Component {
    constructor(props) {
      super(props);

      this.state = {
        formValues: {...initialState},
      };
    }

    handleChange = e => {
      const {name, value} = e.target;
      const {formValues} = this.state;

      formValues[name] = value;
      this.setState({formValues});
    };

    handleSubmit = _handleSubmit => e => {
      e.preventDefault();
      const {formValues} = this.state;

      _handleSubmit(formValues);
    };

    getInputProps = (props = {}) => ({
      onChange: callAll(props.onChange, this.handleChange),
    });

    getStateAndHelpers = () => ({
      formValues: this.state.formValues,
      handleSubmit: this.handleSubmit,
      getInputProps: this.getInputProps,
    });

    render() {
      return (
        <FormComponent
          {...this.props}
          handleSubmit={this.handleSubmit}
          {...this.getStateAndHelpers()}
        />
      );
    }
  };
```

En el form como tal aplicamos la función que extiende, en este caso es un mensaje de log.

```jsx
import {withControlledForm} from '../hocs/with-controlled-form';

const Form = ({formValues, getInputProps, handleSubmit, onSubmit}) => {
  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <p>Name</p>
        <input
          type="text"
          name="name"
          value={formValues.name}
          {...getInputProps({
            onChange: () => console.log('changed'),
          })}
        />
      </div>
      <div>
        <p>Job Title</p>
        <input
          type="text"
          name="jobTitle"
          value={formValues.jobTitle}
          {...getInputProps({
            onChange: () => console.log('changed 2'),
          })}
        />
      </div>
      <button style={{margin: '8px 0'}} type="submit">
        Submit
      </button>
    </form>
  );
};

export const FormWithHoc = withControlledForm(Form, {
  name: '',
  jobTitle: '',
});
```

**Uso de Props Getter con custom hooks**

En este caso mismo procedimiento, se refactoriza el custom hook agregandole la función callAll y la función input props.

```tsx
function callAll(...fns) {
  return function (...args) {
    fns.forEach(fn => fn && fn(...args));
  };
}

export const useControlledForm = initialState => {
  const [formValues, setFormValues] = useState({...initialState});

  const handleChange = ({target}) => {
    const {name, value} = target;

    setFormValues({...formValues, [name]: value});
  };

  const handleSubmit = _handleSubmit => event => {
    event.preventDefault();

    _handleSubmit(formValues);
  };

  const getInputProps = (props = {}) => ({
    onChange: callAll(props.onChange, handleChange),
  });

  const getStateAndHelpers = () => ({
    formValues,
    handleSubmit,
    getInputProps,
  });

  return {
    handleSubmit,
    ...getStateAndHelpers(),
  };
};
```

Y la correspondiente implementación en el componente

```jsx
import {useControlledForm} from '../hooks/useControlledForm';

export const FormWithHook = ({onSubmit}) => {
  const {formValues, getInputProps, handleSubmit} = useControlledForm({
    name: '',
    jobTitle: '',
  });

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <p>Name</p>
        <input
          type="text"
          name="name"
          value={formValues.name}
          {...getInputProps({
            onChange: () => console.log('updated'),
          })}
        />
      </div>
      <div>
        <p>Job Title</p>
        <input
          type="text"
          name="jobTitle"
          value={formValues.jobTitle}
          {...getInputProps()}
        />
      </div>
      <button style={{margin: '8px 0'}} type="submit">
        Submit
      </button>
    </form>
  );
};
```

Al ser la implementación mas limpia, más legible y con menos lineas de código es recomendable usar la implementación con custom hooks, pero esto siempre depende igualmente del caso.

**Links de interes:**

https://kentcdodds.com/blog/how-to-give-rendering-control-to-users-with-prop-getters (Explicación del patrón por su creador)

Ejemplos de liberías que implementan el patrón

https://github.com/downshift-js/downshift#prop-getters

https://www.npmjs.com/package/mm-react-credit-card-primitives

https://www.npmjs.com/package/react-stepper-primitive

## Pattern: State Initializer

Este patrón Consiste en poder darle la opción a un componente que su estado sea inicializado por un componente padre, permitiendonos también manipular o resetear el estado.

**Ventajas**

- Sencillo de implementar
- Iniciar un componente con un estado particular

**Desventajas**

- Lo único es que la implementación va en el padre.

**En que casos aplica usar este patrón?**

Cuando queramos que un componente tenga un estado particular al inicio.

**Ejemplos de aplicación**

La esencia del patrón es enviar como parámetro el estado inicial, en este caso lo haremos con un custom hook

Custom Hook

```jsx
export const useControlledForm = (initialState = {}) => {
  const [formValues, setFormValues] = useState(initialState);

  const handleChange = ({target}) => {
    const {name, value} = target;
    setFormValues({...initialState, [name]: value});
  };

  const handleSubmit = cb => e => {
    e.preventDefault();
    cb(formValues);
  };

  const resetForm = () => {
    setFormValues(initialState);
  };

  return {
    formValues,
    handleChange,
    handleSubmit,
    resetForm,
  };
};
```

Implementación desde el componente

```jsx
export const MyForm = () => {
  const {formValues, handleChange, handleSubmit, resetForm} = useControlledForm(
    {
      name: '',
    },
  );

  const showData = values => {
    alert(JSON.stringify(values));
    resetForm();
  };

  return (
    <form onSubmit={handleSubmit(showData)}>
      <input name="name" value={formValues.name} onChange={handleChange} />
      <button>Submit</button>
    </form>
  );
}
```



## Pattern: State Reducer

Este patrón consiste en delegar el máximo contro de como se va actualizar el estado interno de un componente o un custom hook.

**Ventajas**

- Tenemos el máximo control del estado de un componente

**Desventajas**

- Implementación compleja

**Ejemplos de aplicación**

Básicamente hay que aplicarlo a componentes o custom hooks que funcionan de una manera muy compleja, y que necesitamos que tengan flexibilidad.

**Ejemplos sin patrón**

Usamos un custom hook que manipularía los estados de un player

```jsx
export const usePlayer = () => {
  const [isPlaying, setPlaying] = useState(false);

  const tooglePlay = () => setPlaying(!isPlaying);
  const play = () => setPlaying(true);
  const pause = () => setPlaying(false);

  return {
    tooglePlay,
    isPlaying,
    play,
    pause,
  };
};
```

Componente que lo utiliza

```jsx
export const Player = () => {
  const [count, setCount] = useState(0);
  const {tooglePlay, play, pause, isPlaying} = usePlayer();

  const handlePlay = () => {
    setCount(count + 1);
    play();
  };

  const clickedMoreTimes = () => count >= LIMIT_TIMES;

  return (
    <div className="App">
      <h1>Ejemplo de State Reducer Pattern</h1>
      <p>you clicked {count} times!</p>
      <p>
        Current status: <b>{isPlaying ? 'playing' : 'paused'}</b>
      </p>
      <button disabled={clickedMoreTimes()} onClick={handlePlay}>
        Start
      </button>
      <button onClick={pause}>Pause</button>
      <button onClick={tooglePlay}>Toogle</button>
    </div>
  );
};
```

Un problema podría ser si cambia el requerimiento y nos dicen por ejemplo que la funcionalidad sería limitar los plays a 5, podríamos adaptar la lógica en el custom hook pero le estaríamos dando probablemente mas responsabilidades de las que debería tener el hook.

La justificación para usar el patrón es que el manejo del limite de plays sea lo mas personalizable posible.

**Ejemplos con patrón**

Modificamos el hook, utilizando el react hook useReducer, que sirve para manejar estados complejos.

playerReducer va a ser nuestra función reductora, recibe el estado y la acción a realizar, 

```jsx
export const playerReducer = (state, action) => {
  if (action.type === actionTypes.TOOGLE_PLAYING) {
    return {isPlaying: !state.isPlaying};
  }

  if (action.type === actionTypes.PLAY) {
    return {isPlaying: true};
  }

  if (action.type === actionTypes.PAUSE) {
    return {isPlaying: false};
  }

  return state;
};
```

A useReducer le pasaremos la función reductora y el estado inicial, y vamos a recibir el estado retornado en isPlaying, y la función dispatch, que será la encargada de mutar el estado, 

Cada vez que se llame a dispatch, nuestro reducer va a ser llamado.

```jsx
const [{isPlaying}, dispatch] = useReducer(reducer, {isPlaying: false});
```

Quedando así finalmente nuestro usePlayerReducer custom hook.

```jsx
export const playerReducer = (state, action) => {
  if (action.type === actionTypes.TOOGLE_PLAYING) {
    return {isPlaying: !state.isPlaying};
  }

  if (action.type === actionTypes.PLAY) {
    return {isPlaying: true};
  }

  if (action.type === actionTypes.PAUSE) {
    return {isPlaying: false};
  }

  return state;
};

export const usePlayerReducer = ({reducer = playerReducer} = {}) => {
  const [{isPlaying}, dispatch] = useReducer(reducer, {isPlaying: false});

  const tooglePlay = () => dispatch({type: actionTypes.TOOGLE_PLAYING});
  const play = () => dispatch({type: actionTypes.PLAY});
  const pause = () => dispatch({type: actionTypes.PAUSE});

  return {
    tooglePlay,
    isPlaying,
    play,
    pause,
  };
};

export {actionTypes};
```

En el componente lo que vamos a hacer es enviar una propia función reductora a useReducer, en la cual esté extendiendo la función reductora por defecto del custom Hook, de esta forma no se ve modificada en su lógica y es extensible.

```jsx
import {useState} from 'react';

import {
  usePlayerReducer,
  actionTypes,
  playerReducer,
} from '../hooks/use-player-reducer';

const LIMIT_TIMES = 5;

export const PlayerReducer = () => {
  const [count, setCount] = useState(0);
  const clickedMoreTimes = () => count >= LIMIT_TIMES;

  const {tooglePlay, play, pause, isPlaying} = usePlayerReducer({
    reducer(state, action) {
      const stateUpdated = playerReducer(state, action);

      if (action.type === actionTypes.PLAY && clickedMoreTimes()) {
        alert('limit reached');
        return {
          isPlaying: false,
        };
      }

      return stateUpdated;
    },
  });

  const handlePlay = () => {
    setCount(count + 1);
    play();
  };

  return (
    <div className="App">
      <h1>Ejemplo de State Reducer Pattern</h1>
      <p>you clicked {count} times!</p>
      <p>
        Current status: <b>{isPlaying ? 'playing' : 'paused'}</b>
      </p>
      <button disabled={clickedMoreTimes()} onClick={handlePlay}>
        Start
      </button>
      <button onClick={pause}>Pause</button>
      <button onClick={tooglePlay}>Toogle</button>
    </div>
  );
};
```

**Links de interés**

https://github.com/downshift-js/downshift (implementa el patrón State Reducer)



