<script>
  import * as d3 from "d3"

  export let title = ""
  // export let numbers = ""

  let data = [{anio:2015, cantidad:24}, {anio:2016, cantidad:33}, {anio:2017, cantidad:42}, {anio:2018, cantidad:54}, {anio:2019, cantidad:63}, {anio:2020, cantidad:71}, {anio:2021, cantidad:77}, {anio:2022, cantidad:87}, {anio:2023, cantidad:92}, {anio:2024, cantidad:98}]

  /* 
    d3.scaleLinear() retorna una función,
    la guardamos en alturaColCoffe
    para poder usarla en el template
  */
 let alturaColCoffe = d3.scaleLinear()
 .domain([0, d3.max(data, d => d.cantidad)])
 .range([0, 82]) // ajuste para la tapa del vaso
 
 /* 
   d3.scaleLinear() puede ser usada para
   asignar colores a los elementos,
   la guardamos en color
   para poder usarla en el template
 */
  let color =  d3.scaleLinear()
      .domain(d3.extent(data, d => d.cantidad)) // extent retorna [min, max] de cantidad
      .range(["#EADBC8", "#543310"])

</script>

<h3 class="headline">{title}</h3>

<div class="coffes-container">
  {#each data as d}
  <div>
    <p class="number" style="color: {color(d.cantidad)}">{d.cantidad}</p>
    <div class="column-wrapper">
      <div class="column-coffe" style="height: {alturaColCoffe(d.cantidad)}%; background-color: {color(d.cantidad)}"></div>
      <img class="coffe" src="./images/coffe-cup-mask.svg" alt="">
    </div>
    <p class="anio">{d.anio}</p>
  </div>
  {/each}
</div>

<style>
  .coffes-container {
    display: flex;
    gap: 15px;
  }
  .column-wrapper {
    position: relative;
    width: 100px;
    height: 125px;
    /* background-color: #ccc;
    border: black 1px solid; */
  }
  .column-coffe {
    position: absolute;
    height: 50%;
    left: 10%;
    right: 10%;
    bottom: 0;
    /* width: 100%; */
    background-color: red;
  }
  .coffe {
    position: absolute;
    bottom: -3px;
    width: 100%;
  }
  .number {
    text-align: center;
    font-size: 20px;
    font-weight: 900;
    margin-bottom: 15px;
  }
  .anio {
    text-align: center;
    font-size: 14px;
    margin-top: 15px;
  }
</style>
