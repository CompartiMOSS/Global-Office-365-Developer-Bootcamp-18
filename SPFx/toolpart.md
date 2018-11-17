## Propiedades de webparts


### Organización

A la hora de desarrollar un webpart en Spfx, una parte muy importante del mismo serán las propiedades de configuración que definiremos en la propia clase del webpart.

Para asignar valores a las propiedades lo haremos a través de controles que definiremos en el panel de propiedades.

A la hora de definir el panel, no sólo tendremos que indicar qué controles están vinculados a las propiedades, si no que además podremos organizar dichos controles en páginas y grupos. Esta organización será de gran ayuda en caso de que tengamos muchas propiedades y/o podamos agrupar las propiedades en grupos claramente definidos.

En este ejemplo vamos a mostrar como configurar el panel de propiedades y organizar las distintas propiedades en páginas y grupos, así mismo mostraremos diferentes tipos de controles.

En primer lugar, a través de yeoman nos crearemos una nueva solución de webpart con framework React.

A continuación en la propia clase del webpart buscamos el método "getPropertyPaneConfiguration()" que por defecto aparecerá así:

```js
  protected getPropertyPaneConfiguration():PropertyPaneConfiguration {
    return {
      pages: [
        {
          header: {
            description: strings.PropertyPaneDescription
          },
          groups: [
            {
              groupName: strings.BasicGroupName,
              groupFields: [
                PropertyPaneTextField('description', {
                  label: strings.DescriptionFieldLabel
                })
              ]
            }
          ]
        }
      ]
    };
  }
```


Como vemos el método devuelve un objeto con una propiedad denominada "pages" la cual es un array de objetos donde definiremos cada página del panel de propiedades.

A continuación sustituimos el código de la página de controles por defecto por el siguiente para añadir nuestra primera página personalizada de controles.

```js
{
          header: {
            description: strings.FirstPageHeader
          },
          groups: [
            {
              groupName: strings.FirstGroupName,
              groupFields: [
                PropertyPaneTextField('title', 
                {label: strings.TitleFieldLabel}) ,
                PropertyPaneTextField('subtitle', 
                {label: strings.SubtitleFieldLabel}),
                PropertyPaneTextField('url', 
                {label: strings.UrlFieldLabel})
              ]
            }, {
              groupName: strings.SecondGroupName,
              groupFields: [
                PropertyPaneCheckbox('isModern', {text: strings.IsModernFieldLabel}),
                  PropertyPaneTextField('htmlView', {
                  label: strings.HtmlViewFieldLabel,
                  multiline: true
                })]
            }
          ]
        }
```
El texto que aparecerá de encabezado en la página lo definimos a través del objeto "header".

Así mismo, a través de la colección "groups" podemos agrupar los controles de una misma página en distintos grupos.

En el ejemplo concreto hemos definido una página con dos grupos de controles. El primer grupo se compone de tres propiedades de tipo textbox y una de tipo checkbox. El segundo de una propiedad de tipo textbox multiline.

Este tipo de controles de propiedad, tanto el PropertyPaneTextField como PropertyPaneCheckbox son controles que proporciona Spfx por defecto y que podremos usar con su correspondiente "Import". En este caso concreto sustituiremos:
```js
import {
  BaseClientSideWebPart,
  IPropertyPaneConfiguration,
  PropertyPaneTextField
} from '@microsoft/sp-webpart-base';
```
por
```js
import {
    BaseClientSideWebPart, 
    IPropertyPaneConfiguration, 
    PropertyPaneTextField, 
    PropertyPaneCheckbox,
    PropertyPaneSlider
} from '@microsoft/sp-webpart-base';
```

Así mismo, actualizaremos el archivo de literales y su traducción para que contenga todos los literales que usaremos a lo largo del workshop.

**mystrings.d.ts**
```js
declare interface IPropertiesSampleWebPartStrings {
  FirstPageHeader: string;
  SecondPageHeader: string;
  ThirdPageHeader: string;
  FirstGroupName: string;
  SecondGroupName: string;
  ThirdGroupName: string;
  TitleFieldLabel: string;
  SubtitleFieldLabel: string;
  UrlFieldLabel: string;
  IsModernFieldLabel: string;
  HtmlViewFieldLabel: string;
  OffSetFieldLabel: string;
}

declare module 'PropertiesSampleWebPartStrings' {
  const strings: IPropertiesSampleWebPartStrings;
  export = strings;
}
```

**en-us.js**
```js
define([], function() {
  return {
    "FirstPageHeader": "First property page",
    "SecondPageHeader": "Second property page",
    "ThirdPageHeader": "Third property page",
    "FirstGroupName": "First property group",
    "SecondGroupName": "Second property group",
    "ThirdGroupName": "Third property group",
    "TitleFieldLabel": "Title",
    "SubtitleFieldLabel": "Subtitle",
    "UrlFieldLabel":"Url",
    "IsModernFieldLabel": "Is modern site?",
    "HtmlViewFieldLabel": "Html view",
    "OffSetFieldLabel": "Offset"
  }
});
```

Una vez apliquemos estos cambios, lanzamos un "gulp serve", agregamos el webpart y lo que tendremos es una primera página de propiedades denominada "First property page" que incluye dos grupos de propiedades "First property group" y "Second property group" que incluyen varios tipos distintos.


## Validaciones

Centremonos en las tres primeras propiedades, para empezar vamos a modificar el webpart para que aparezca en el webpart el valor que almacenan dichas propiedades.

En primer lugar sustituimos el interfaz que define las props del webpart:

**IPropertiesSampleProps.ts**
```js
export interface IPropertiesSampleWebPartProps {
  description: string;
}
```
por
```js
export interface IPropertiesSampleWebPartProps {
  title : string;
  subTitle : string;
  url : string;
  isModern : boolean;
  htmlView : string;
  offset : string;
  collectionData : any[];
}
```


Este interfaz define el objeto donde el webpart almacena los valores de las propiedades que el usuario inserta en el panel de propiedades. A través de este objeto podemos pasar dichos valores a los componentes de React que componen nuestro webpart.

Modificamos también el componente principal para que muestre dichos valores. 
Actualizamos las props del componente:

**IPropertiesSampleProps.ts**
```js
export interface IPropertiesSampleProps {
  title: string;
  subtitle: string;
  url: string;  
}
```

Actualizamos el método render del webpart:
**PropertiesSampleWebPart.ts**
```js
  public render() : void {
    const element: React.ReactElement < IPropertiesSampleProps > = React.createElement(PropertiesSample, 
      {
        title: this.properties.title,
        subtitle: this.properties.subtitle,
        url:this.properties.url
      });

    ReactDom.render(element, this.domElement);
  }
```

Y el método render del componente principal:
**PropertiesSampleWebPart.ts**
```js
  public render(): React.ReactElement < IPropertiesSampleProps > {
    return(
      <div className={styles.propertiesSample}>
        <div className={styles.container}>
          <div className={styles.row}>
            <div className={styles.column}>
              <span className={styles.title}>Welcome to Global Office 365 Developer Bootcam!</span>
              <p className={styles.subTitle}>Customizing property pane with custom validations.</p>
              <p className={styles.description}>{escape("Title: " + this.props.title)}</p>
              <p className={styles.description}>{escape("Subtitle: " + this.props.subtitle)}</p>
              <p className={styles.description}>{escape("Url: " + this.props.url)}</p>
              <a href="https://aka.ms/spfx" className={styles.button}>
                <span className={styles.label}>Learn more</span>
              </a>
            </div>
          </div>
        </div>
      </div>
    );
  }
```

Si volvemos a compilar el webpart, veremos que en pantalla se muestran los valores de las tres primeras propiedades.


## Validaciones
Hasta ahora lo que hemos visto es sencillo, pero qué ocurre si queremos que en dichas propiedades existan validaciones, por ejemplo que una de ellas sea obligatoria el rellenarla, o que sea un email, o que tenga una longitud máxima, etc...

En el caso de los "PropertyPaneTextField" disponemos de una propiedad que nos permitirá incluir validación: "onGetErrorMessage".

Vamos a empezar incluyendo varias validaciones distintas en nuestros PropertyPaneTextField" del primer grupo.
Incluiremos los siguientes métodos dentro de la clase webpart:

```js
 private requireValidation(value: string): string {
    if (value === null || value.trim().length === 0) {
        return "Required. ";
    }
    return "";
 }
 private maxLengthValidation(value: string): string {
            if (value.length > 20) {
                return "Field should not be longer than 20 characters. ";
            }
            return "";
 }
```

Así mismo, para aplicar estas validaciones modificaremos nuestro primer grupo de propiedades con el siguiente código:

```js
[
    PropertyPaneTextField('title', 
    {label: strings.TitleFieldLabel,
    onGetErrorMessage: this.requireValidation.bind(this)}) ,
    PropertyPaneTextField('subtitle', 
    {label: strings.SubtitleFieldLabel,
    onGetErrorMessage: this.maxLengthValidation.bind(this)}),
    PropertyPaneTextField('url', 
    {label: strings.UrlFieldLabel,
    onGetErrorMessage: this.requireValidation.bind(this)})                
]
```

## El problema


Lo que hemos visto hasta ahora es sencillo y nos permite aplicar validaciones de manera rápida sobre nuestras propiedades.

Si tenemos muchas propiedades iremos definiendo las diferentes funciones que harán de validación y las iremos aplicando a cada propiedad según corresponda. 

Pero y si en nuestra solución tenemos varios webparts cuyas propiedades usan las mismas validaciones...¿repetiremos código por toda la solución? Eso todos sabemos que no es una buena práctica. Y si varias de nuestras propiedades combinan varias validaciones a la vez, ¿creamos nuevas validaciones combinando otras repitiendo el código?

A partir de aquí veremos un ejemplo real de cómo por una parte centralizar todas las validaciones en un solo punto y además cómo podemos combinarlas sin tener código repetido por nuestra solución.


## La solución


Para poder conseguir nuestro objetivo tenemos dos opciones, la primera sería crearnos nuestro custom control implementando su propia funcionalidad de validación, y la segunda que es coger un control que ya existe y extender su funcionalidad para que incorpore la parte de validación.

En nuestro caso hemos optado por la segunda, a partir del "PropertyPaneTextField" nos crearemos nuestro propio "PropertyPaneTextFieldWithMultipleValidations".

Para empezar creamos una carpeta llamada "utils" bajo la carpeta "webparts" e incluimos el siguiente archivo:

**PropertyPaneTextFieldWithValidations.ts**
```js
import {PropertyPaneTextField, IPropertyPaneTextFieldProps, IPropertyPaneField} from "@microsoft/sp-webpart-base";

export enum ValidationType {
    required = 1,
    maxLength = 2,
    validUrl = 3,
    minLength = 4
}

export interface IValidationTypeWithParameters {
    validationType : ValidationType;
    parameters? : any;
}

export function PropertyPaneTextFieldWithValidations(targetProperty : string, properties : IPropertyPaneTextFieldProps, validationType : IValidationTypeWithParameters) : IPropertyPaneField < IPropertyPaneTextFieldProps > {
    let validations = new Validations();
    let validation: (value : string) => string = validations.GetValidation(validationType);
    return PropertyPaneTextField(targetProperty, {
        ...properties,
        onGetErrorMessage: validation
    });
}

class Validations {
    public GetValidation(validationType : IValidationTypeWithParameters) : (value: string) => string {
        switch(validationType.validationType) {
            case ValidationType.required:
                return this.GetRequiredValidation();
            case ValidationType.maxLength:
                return this.GetMaxLengthValidation(validationType.parameters);
            case ValidationType.validUrl:
                return this.GetUrlValidation();
            case ValidationType.minLength:
                return this.GetMinLengthValidation(validationType.parameters);
        }
    }

    private GetMaxLengthValidation(maxLength:number) : (value : string) => string {
        return (value: string): string => {
            if (value.length > maxLength) {
                return "Field should not be longer than " + maxLength + " characters. ";
            }
            return "";
        };
    }

    private GetMinLengthValidation(minLength:number) : (value : string) => string {
        return(value : string): string => {
            if (value.length < minLength) {
                return "Field should be longer than " + minLength + " characters. ";
            }
            return "";
        };
    }

    private GetUrlValidation() : (value : string) => string {
        return(value : string): string => {
            let pattern : RegExp = /^(?:http(s)?:\/\/)?[\w.-]+(?:\.[\w\.-]+)+[\w\-\._~:/?#[\]@!\$&'\(\)\*\+,;=.]+$/gi;
            if (!pattern.test(value)) {
                return "Please enter a valid URL. ";
            }
            return "";
        };
    }

    private GetRequiredValidation(): (value: string) => string {
        return (value: string): string => {
            if (value === null || value.trim().length === 0) {
                return "Required. ";
            }
            return "";
        };
    } 
}
```

Básicamente lo que estamos haciendo es tener una clase que es el único punto de acceso donde obtener las funciones de validación para toda la solución.

Estamos implementando una función distinta para cada tipo de validación. A medida que queramos incorporar nuevas validaciones únicamente tendremos que crear un nuevo método que la implemente, crear un nuevo valor en la enumeración "ValidationType" y por último modificar el método "GetValidation" para incorporar el nuevo tipo.

En el archivo también hemos definido el nuevo control que usaremos para incorporar esta nueva funcionalidad a nuestras propiedades. 

Sustituiremos nuestros anteriores "PropertyPaneTextField" por nuestros "PropertyPaneTextFieldWithMultipleValidations" en el webpart indicando que validación queremos que aplique, para ello incorporaremos en primer lugar el "import" correspondiente:

```js
import {
  ValidationType,
  PropertyPaneTextFieldWithValidations
} from '../utils/PropertyPaneTextFieldWithValidations';
```

Además reemplazaremos nuestro anterior grupo de propiedades por el siguiente:
```js
[
    PropertyPaneTextFieldWithValidations('title', 
    {label: strings.TitleFieldLabel},
    {validationType: ValidationType.required}) ,
    PropertyPaneTextFieldWithValidations('subtitle', 
    {label: strings.SubtitleFieldLabel},
    {validationType: ValidationType.maxLength,parameters:10}),
    PropertyPaneTextFieldWithValidations('url', 
    {label: strings.UrlFieldLabel},
    {validationType: ValidationType.required})                
]
```         
Volvemos a compilar y vemos que ya tenemos funcionando las validaciones con nuestros nuevos controles.

¿Pero y qué pasa con lo de combinar las validaciones? Tranquilos, no nos hemos olvidado...vais a comprobar que con un pequeño cambio tendremos controles donde podremos definir varias validaciones para aplicar.

En la clase "Validations" en el archivo "PropertyPaneTextFieldWithValidations.ts" incluiremos el siguiente método:
```js
    public GetMultipleValidations(validationTypes: IValidationTypeWithParameters[]): (value: string) => string {
        return (value: string): string => {
            let result: string = "";
            for (let validationType of validationTypes) {
                result += this.GetValidation(validationType)(value);
            }
            return result;
        };
    }
``` 
En el mismo archivo sustituimos el anterior control por este nuevo:
```js
export function PropertyPaneTextFieldWithMultipleValidations(targetProperty : string, properties : IPropertyPaneTextFieldProps, validationTypes : IValidationTypeWithParameters[]) : IPropertyPaneField < IPropertyPaneTextFieldProps > {
    let validations = new Validations();
    let validation: (value : string) => string = validations.GetMultipleValidations(validationTypes);
    return PropertyPaneTextField(targetProperty, {
        ...properties,
        onGetErrorMessage: validation
    });
}
``` 

Y cambiaremos nuestro grupo de propiedades en el webpart para llamar a este nuevo control:
```js
[
    PropertyPaneTextFieldWithMultipleValidations('title', 
    {label: strings.TitleFieldLabel},
        [
        { validationType: ValidationType.required },
        { validationType: ValidationType.maxLength, parameters: 10 }
        ]),
    PropertyPaneTextFieldWithMultipleValidations('subtitle', 
    {label: strings.SubtitleFieldLabel},                
    [
        { validationType: ValidationType.required },
        { validationType: ValidationType.minLength, parameters: 3 }
    ]),
    PropertyPaneTextFieldWithMultipleValidations('url', 
    {label: strings.UrlFieldLabel},                
    [
        { validationType: ValidationType.required },
        { validationType: ValidationType.validUrl }
    ])                
]
``` 

Volvemos a compilar con los últimos cambios y podemos probar a ver cómo están aplicando todas las validaciones sobre nuestras propiedades.

A partir de aquí, os invito a incorporar nuevas validaciones que se os ocurran, y si os gusta el riesgo a probar con validaciones asíncronas.

Por último comentaros que el siguiente paso natural es que una vez desarrollado este nuevo control y este conjunto de validaciones, lo siguiente sería generar un nuevo paquete npm en vuestro repositorio privado para que estuviera disponible para todo vuestro equipo y para las futuras soluciones que vayais desarrollando.

Happy coding!!!
