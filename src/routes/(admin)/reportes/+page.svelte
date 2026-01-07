<!-- src/routes/(admin)/reportes/+page.svelte -->
<script>
  import { BarChart3, Download, Calendar, Filter, TrendingUp, Package, Users, DollarSign } from 'lucide-svelte';
  import FiltrosReportes from '$lib/components/reportes/FiltrosReportes.svelte';
  import EstadisticasCards from '$lib/components/reportes/EstadisticasCards.svelte';
  import TablaReporte from '$lib/components/reportes/TablaReporte.svelte';
  import ExportarExcel from '$lib/components/reportes/ExportarExcel.svelte';
  
  export let data;
  
  let filtrosAplicados = {};
  let datosReporte = [];
  let estadisticas = data.estadisticasIniciales;
  let cargando = false;
  let mostrarFiltros = true;
  
  // Función para cargar datos con filtros
  async function cargarReporte(filtros) {
    cargando = true;
    filtrosAplicados = filtros;
    
    try {
      const params = new URLSearchParams();
      
      // Construir query params
      if (filtros.fechaInicio) params.append('fechaInicio', filtros.fechaInicio);
      if (filtros.fechaFin) params.append('fechaFin', filtros.fechaFin);
      if (filtros.estados?.length) params.append('estados', filtros.estados.join(','));
      if (filtros.categorias?.length) params.append('categorias', filtros.categorias.join(','));
      if (filtros.marcas?.length) params.append('marcas', filtros.marcas.join(','));
      if (filtros.telefono) params.append('telefono', filtros.telefono);
      if (filtros.productoId) params.append('productoId', filtros.productoId);
      
      const response = await fetch(`/api/reportes?${params.toString()}`);
      const result = await response.json();
      
      if (result.success) {
        datosReporte = result.data.pedidos || [];
        estadisticas = result.data.estadisticas || {};
      }
    } catch (error) {
      console.error('Error cargando reporte:', error);
    } finally {
      cargando = false;
    }
  }
  
  function handleFiltrosChange(event) {
    cargarReporte(event.detail);
  }
</script>

<svelte:head>
  <title>Reportes y Análisis | Dashboard</title>
</svelte:head>

<div class="max-w-7xl mx-auto space-y-6">
  
  <!-- Header -->
  <div class="flex flex-col md:flex-row md:items-center md:justify-between gap-4">
    <div>
      <h1 class="text-2xl md:text-3xl font-bold text-gray-900 flex items-center gap-2">
        <BarChart3 class="w-8 h-8 text-primary-600" />
        Reportes y Análisis
      </h1>
      <p class="mt-1 text-sm text-gray-600">
        Genera reportes personalizados y exporta tus datos a Excel
      </p>
    </div>
    
    <div class="flex items-center gap-3">
      <button
        on:click={() => mostrarFiltros = !mostrarFiltros}
        class="btn-secondary flex items-center gap-2"
      >
        <Filter class="w-4 h-4" />
        {mostrarFiltros ? 'Ocultar' : 'Mostrar'} Filtros
      </button>
      
      <ExportarExcel 
        {datosReporte} 
        {estadisticas} 
        {filtrosAplicados}
      />
    </div>
  </div>

  <!-- Filtros -->
  {#if mostrarFiltros}
    <div class="bg-white rounded-xl shadow-sm border border-gray-200 p-6">
      <FiltrosReportes
        categorias={data.categorias}
        marcas={data.marcas}
        estados={data.estados}
        on:aplicar={handleFiltrosChange}
      />
    </div>
  {/if}

  <!-- Estadísticas -->
  {#if estadisticas}
    <EstadisticasCards {estadisticas} {cargando} />
  {/if}

  <!-- Tabla de Resultados -->
  <div class="bg-white rounded-xl shadow-sm border border-gray-200">
    <div class="p-6 border-b border-gray-200">
      <div class="flex items-center justify-between">
        <h2 class="text-lg font-semibold text-gray-900">
          Resultados del Reporte
        </h2>
        <span class="text-sm text-gray-600">
          {datosReporte.length} pedido{datosReporte.length !== 1 ? 's' : ''}
        </span>
      </div>
    </div>
    
    <TablaReporte datos={datosReporte} {cargando} />
  </div>

  <!-- Reportes Rápidos -->
  <div class="bg-gradient-to-r from-primary-50 to-primary-100 rounded-xl p-6 border border-primary-200">
    <h3 class="text-lg font-semibold text-gray-900 mb-4 flex items-center gap-2">
      <TrendingUp class="w-5 h-5 text-primary-600" />
      Reportes Rápidos
    </h3>
    
    <div class="grid grid-cols-2 md:grid-cols-4 gap-3">
      <button
        on:click={() => cargarReporte({ preset: 'hoy' })}
        class="btn-outline bg-white hover:bg-primary-50 flex items-center justify-center gap-2"
      >
        <Calendar class="w-4 h-4" />
        Hoy
      </button>
      
      <button
        on:click={() => cargarReporte({ preset: 'semana' })}
        class="btn-outline bg-white hover:bg-primary-50 flex items-center justify-center gap-2"
      >
        <Calendar class="w-4 h-4" />
        Esta Semana
      </button>
      
      <button
        on:click={() => cargarReporte({ preset: 'mes' })}
        class="btn-outline bg-white hover:bg-primary-50 flex items-center justify-center gap-2"
      >
        <Calendar class="w-4 h-4" />
        Este Mes
      </button>
      
      <button
        on:click={() => cargarReporte({ estados: ['pendiente'] })}
        class="btn-outline bg-white hover:bg-primary-50 flex items-center justify-center gap-2"
      >
        <Package class="w-4 h-4" />
        Pendientes
      </button>
    </div>
  </div>

  <!-- Info de ayuda -->
  <div class="bg-blue-50 border border-blue-200 rounded-lg p-4">
    <div class="flex items-start gap-3">
      <svg class="w-5 h-5 text-blue-600 flex-shrink-0 mt-0.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"/>
      </svg>
      <div class="text-sm text-blue-800">
        <p class="font-medium mb-1">💡 Consejos de uso:</p>
        <ul class="list-disc list-inside space-y-1 text-blue-700">
          <li>Usa los filtros para personalizar tu reporte</li>
          <li>Los "Reportes Rápidos" te dan acceso instantáneo a datos comunes</li>
          <li>Exporta a Excel para análisis más detallados</li>
          <li>Los datos se actualizan en tiempo real</li>
        </ul>
      </div>
    </div>
  </div>

</div>