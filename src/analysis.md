---
title: SysArmy analisis
toc: false
index: 0
---

```js
const data = await FileAttachment("data/sysArmyModified.csv").csv({typed: true, header: true});
```

```js
const salarioPorSeniority = Object.entries(
  data.reduce((acc, curr) => {
    if (curr.salario_neto && curr.seniority) { 
      if (!acc[curr.seniority]) {
        acc[curr.seniority] = { totalSalario: 0, count: 0 };
      }
      acc[curr.seniority].totalSalario += curr.salario_neto;
      acc[curr.seniority].count += 1;
    }
    return acc;
  }, {})
).map(([seniority, { totalSalario, count }]) => ({
  seniority,
  promedio: totalSalario / count,
  count
}));

salarioPorSeniority.sort((a, b) => b.count - a.count);

function plotSalarioPorSeniority() {
  return Plot.plot({
    title: "Salario Promedio según Seniority",
    width: 1000,
    height: 500,
    marginLeft: 100,
    x: {
      label: "Promedio de Salario Neto ($)",
    },
    y: {
      label: "Seniority",
      domain: salarioPorSeniority.map(d => d.seniority),
    },
    marks: [
      Plot.barX(salarioPorSeniority, {
        x: "promedio",   
        y: "seniority",  
        fill: "steelblue",
        title: d => `${d.seniority}: ${d.promedio.toFixed(2)}`,
        tip: d => `${d.seniority}: ${d.promedio.toFixed(2)}`,  
      }),
    ],
  });
}

```

```js
function empleosPorModalidad() {
const modalidadCount = data.reduce((acc, curr) => {
  if (curr.modalidad) {
  
    const modalidad = curr.modalidad === "100% remoto" ? "Remoto" :
                      curr.modalidad === "100% presencial" ? "Presencial" :
                      curr.modalidad === "Híbrido (presencial y remoto)" ? "Híbrido" : "Otro";

    acc[modalidad] = (acc[modalidad] || 0) + 1;
  }
  return acc;
}, {});

const modalidadDistribucion = Object.entries(modalidadCount).map(([modalidad, count]) => ({
  modalidad,
  count
}));

return Plot.plot({
  title: "Distribución de Empleos por Modalidad",
  x: {
    label: "Número de Empleos",
  },
  y: {
    label: "Modalidad de Trabajo",
  },
  marks: [
  
    Plot.barY(modalidadDistribucion, {
      x: "modalidad",
      y: "count",
      fill: "steelblue",
      title: d => `${d.modalidad}: ${d.count}`,
      tip: d => `${d.modalidad}: ${d.count}`, 
    }),
  ]
});

}
```


<div class="hero">
  <h1>Analisis SysArmy</h1>
</div>

<div class="card">
  ${plotSalarioPorSeniority()}
</div>

<div class="grid grid-cols-1">
    <h2>Trabajos según modalidad</h2>
</div>

```js
  const selectedModalidad = view(Inputs.select(new Map([
      ["Remoto", "100% remoto"],
      ["Presencial", "100% presencial"],
      ["Híbrido", "Híbrido (presencial y remoto)"],
    ]), {label: "Selecciona una Modalidad"}));

```

```js
  function modalidadPorSeniority() {
  
    const empleadosPresenciales = data.filter(d => d.modalidad === selectedModalidad);

    const adapter = {
      "100% remoto": "Remoto",
      "100% presencial": "Presencial",
      "Híbrido (presencial y remoto)": "Híbrido"
    }


  
    const empleadosPresencialesPorSeniority = empleadosPresenciales.reduce((acc, curr) => {
    
      const seniority = curr.seniority;
      acc[seniority] = (acc[seniority] || 0) + 1;
      return acc;
    }, {});

  
    const empleadosPresencialesDistribucion = Object.entries(empleadosPresencialesPorSeniority).map(([seniority, count]) => ({
      seniority,
      count
    }));

    empleadosPresencialesDistribucion.sort((a,b) => b.count - a.count)

  
    return Plot.plot({
      title: `Cantidad de Empleados ${adapter[selectedModalidad]} por Seniority`,
      width: 1000,
      height: 700,
      marginLeft: 100,
      x: {
        label: "Número de Empleos",
      },
      y: {
        label: "Seniority",
        domain: empleadosPresencialesDistribucion.map(e => e.seniority)
      },
      marks: [
      
        Plot.barX(empleadosPresencialesDistribucion, {
          x: "count",      
          y: "seniority",  
          fill: "steelblue",
          title: d => `${d.seniority}: ${d.count}`,
          tip: d => `${d.seniority}: ${d.count}`, 
        }),
      ]
    });
  }
```
<div class="grid grid-cols-2">
    <div class="card">
      ${empleosPorModalidad()}
    </div>
    <div class="card">
      ${modalidadPorSeniority()}
    </div>
</div>



