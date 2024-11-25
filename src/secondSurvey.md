---
title: Second Survey
toc: false
index: 1
---

```js
const data = await FileAttachment("data/sysArmyModified.csv").csv({typed: true, header: true});
```

```js
function respuestasGuardias() {
  const respuestasGuardias = data.map(d => d.guardias);

  const conteoGuardias = respuestasGuardias.reduce((acc, curr) => {
    const respuesta = curr || "No responde";
    acc[respuesta] = (acc[respuesta] || 0) + 1;
    return acc;
  }, {});

  const categoriasGuardias = ["Sí, activa", "Sí, pasiva", "No", "No responde"];
  categoriasGuardias.forEach(categoria => {
    if (!conteoGuardias[categoria]) {
      conteoGuardias[categoria] = 0; 
    }
  });

  const distribucionGuardias = categoriasGuardias.map(categoria => ({
  respuesta: categoria,
  count: conteoGuardias[categoria]
}));

distribucionGuardias.sort((a,b) => b.count - a.count);


return Plot.plot({
  title: "Cantidad de Personas que Hacen Guardias",
  width: 800, 
  height: 500,
  marginLeft: 100, 
  x: {
    label: "Cantidad de Personas", 
    fontSize: 18, 
  },
  y: {
    label: "Respuestas de Guardia", 
    domain: distribucionGuardias.map((categoria) => categoria.respuesta), 
    fontSize: 18, 
  },
  marks: [
    Plot.barX(distribucionGuardias, {
      x: "count",       
      y: "respuesta",   
      fill: "steelblue", 
      title: d => `${d.respuesta}: ${d.count}`,
      tip: d => `${d.respuesta}: ${d.count}`, 
    }),
  ]
});
  }

```

```js
function respuestasEstudios() {

const respuestasEstudio = data.map(d => d.nivel_de_estudio);

const conteoEstudio = respuestasEstudio.reduce((acc, curr) => {

  const respuesta = (curr && curr.trim()) ? curr : "No responde";
  acc[respuesta] = (acc[respuesta] || 0) + 1;
  return acc;
}, {});

const categoriasEstudio = [
  "Secundario", 
  "Terciario", 
  "Universitario", 
  "Maestría", 
  "Doctorado", 
  "Posgrado/Especialización", 
  "No responde"
];

categoriasEstudio.forEach(categoria => {
  if (!conteoEstudio[categoria]) {
    conteoEstudio[categoria] = 0; 
  }
});

const distribucionEstudio = categoriasEstudio.map(categoria => ({
  respuesta: categoria,
  count: conteoEstudio[categoria]
}));

distribucionEstudio.sort((a, b) => b.count - a.count);

return Plot.plot({
  title: "Cantidad de Empleados por Nivel de Estudio",
  width: 800, 
  height: 500,
  marginLeft: 150, 
  x: {
    label: "Cantidad de Empleados", 
    fontSize: 18, 
  },
  y: {
    label: "Nivel de Estudio", 
    domain: distribucionEstudio.map(d => d.respuesta), 
    fontSize: 18, 
  },
  marks: [
    Plot.barX(distribucionEstudio, {
      x: "count",       
      y: "respuesta",   
      fill: "steelblue", 
      title: d => `${d.respuesta}: ${d.count}`,
      tip: d => `${d.respuesta}: ${d.count}`, 
    }),
  ]
});
}
```



<div class="hero">
  <h1>Second Survey SysArmy</h1>
</div>

<div class="card">
  ${respuestasGuardias()}
</div>

```js
  const options = ["Full-Time", "Part-Time"];
  const selected = view(Inputs.select(options, {label: "Selecciona una modalidad"}));
```


```js
  function sueldoPorNivelDeEstudio() {
  
const datosFiltrados = data.filter(d => d.salario_neto && d.nivel_de_estudio && d.nivel_de_estudio.trim() !== "" && d.dedicacion == selected);


const promedioSueldoPorEstudio = datosFiltrados.reduce((acc, curr) => {
  const nivelEstudio = curr.nivel_de_estudio.trim();
  const salario = curr.salario_neto;

  if (!acc[nivelEstudio]) {
    acc[nivelEstudio] = { totalSalario: 0, count: 0 };
  }

  acc[nivelEstudio].totalSalario += salario;
  acc[nivelEstudio].count += 1;

  return acc;
}, {});

const promedioSueldoDistribucion = Object.entries(promedioSueldoPorEstudio).map(([nivelEstudio, { totalSalario, count }]) => ({
  nivelEstudio,
  promedioSueldo: totalSalario / count,
}));

const promedioSueldoOrdenado = promedioSueldoDistribucion.sort((a, b) => b.promedioSueldo - a.promedioSueldo);

return Plot.plot({
  title: `Promedio de Sueldo ${selected} por Nivel de Estudio`,
  width: 800, 
  height: 500,
  marginLeft: 150, 
  x: {
    label: "Promedio de Sueldo ($)", 
    fontSize: 18, 
  },
  y: {
    label: "Nivel de Estudio", 
    domain: promedioSueldoOrdenado.map(d => d.nivelEstudio), 
    fontSize: 18, 
  },
  marks: [
    Plot.barX(promedioSueldoOrdenado, {
      x: "promedioSueldo",       
      y: "nivelEstudio",         
      fill: "steelblue",         
      title: d => `${d.nivelEstudio}: $${d.promedioSueldo.toFixed(2)}`,
      tip: d => `${d.nivelEstudio}: $${d.promedioSueldo.toFixed(2)}`,  
    }),
  ]
});

  }
```

```js
function cantidadPorFacultad() {
  // Paso 1: Filtrar los datos válidos
const datosConInstitucion = data.filter(d => d.institucion_educativa && d.institucion_educativa.trim() !== "");

// Paso 2: Contar cuántas personas hay por institución educativa
const personasPorInstitucion = datosConInstitucion.reduce((acc, curr) => {
  const institucion = curr.institucion_educativa.trim();

  acc[institucion] = (acc[institucion] || 0) + 1;

  return acc;
}, {});

// Paso 3: Convertir los resultados a un formato adecuado para Plot.js
const distribucionInstituciones = Object.entries(personasPorInstitucion).map(([institucion, count]) => ({
  institucion: institucion.split('-')[0],
  count
}));

// Paso 4: Ordenar las instituciones por número de personas (de mayor a menor)
const institucionesOrdenadas = distribucionInstituciones.sort((a, b) => b.count - a.count).splice(0, 10);

// Paso 5: Crear el gráfico de barras horizontal
return Plot.plot({
  title: "Top 10 Instituciones Educativas con más empleados",
  width: 1000,  // Ajusta el ancho del gráfico
  height: 600,  // Ajusta la altura del gráfico
  marginLeft: 150,  // Margen para las etiquetas largas del eje Y
  x: {
    label: "Número de Personas",  // Eje X: cantidad de personas
  },
  y: {
    label: "Institución Educativa",  // Eje Y: instituciones educativas
    domain: institucionesOrdenadas.map(d => d.institucion),  // Ordenadas por cantidad
  },
  marks: [
    Plot.barX(institucionesOrdenadas, {
      x: "count",       // Eje X: cantidad de personas
      y: "institucion", // Eje Y: instituciones
      fill: "steelblue", // Color de las barras
      title: d => `${d.institucion}: ${d.count}`, // Muestra el conteo al pasar el mouse
      tip: d => `${d.institucion}: ${d.count}`,   // Detalle adicional al pasar el mouse
    }),
  ]
});

}
```
<div class="grid grid-cols-2">
  <div class="card">
    ${respuestasEstudios()}
  </div>
  <div class="card">
    ${sueldoPorNivelDeEstudio()}
  </div>
</div>

<div class="card">
  ${cantidadPorFacultad()}
</div>


