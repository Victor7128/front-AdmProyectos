<script>
  import { onMount } from "svelte";

  const API_BASE_URL = "http://127.0.0.1:8000";

  let isLoading = false;
  let result = null;
  let error = null;
  let isOpen = false;

  $: hasImage =
    typeof window !== "undefined" &&
    sessionStorage?.getItem("uploadedImage") != null;

  // ✅ Filtrar advertencias válidas y separar por tipo
  $: advertencias =
    result && result.advertencias
      ? result.advertencias.filter((a) => !!a && a !== "✅ Sin observaciones")
      : [];

  $: advertenciasErrores = advertencias.filter((a) => a.startsWith("❌"));
  $: advertenciasWarnings = advertencias.filter((a) => a.startsWith("⚠️"));
  $: advertenciasExito = result?.advertencias?.includes("✅ Sin observaciones");

  $: advertenciasCount = advertencias.length;

  // ✅ Guardar en sessionStorage
  $: if (!isLoading && result !== null && typeof window !== "undefined") {
    sessionStorage.setItem(
      "advertencias_ocr",
      advertenciasCount > 0 ? advertenciasCount.toString() : "0",
    );

    // Guardar si es válido (sin errores críticos)
    const esValido = advertenciasErrores.length === 0;
    sessionStorage.setItem("ocr_valido", esValido ? "true" : "false");
  }

  onMount(() => {
    if (hasImage) processImage();
  });

  // Vuelve a procesar si cambia la imagen
  $: if (hasImage && !isLoading && !result && !error) {
    processImage();
  }
  $: if (!hasImage) {
    result = null;
    error = null;
  }

  async function base64ToBlob(base64String) {
    const response = await fetch(base64String);
    return response.blob();
  }

  async function processImage() {
    const uploadedImage = sessionStorage.getItem("uploadedImage");
    if (!uploadedImage) {
      return;
    }

    isLoading = true;
    error = null;
    result = null;

    try {
      const blob = await base64ToBlob(uploadedImage);
      const formData = new FormData();
      formData.append("file", blob, "comprobante.jpg");
      const controller = new AbortController();

      const apiResponse = await fetch(`${API_BASE_URL}/filtro_ocr`, {
        method: "POST",
        body: formData,
        signal: controller.signal,
      });

      if (!apiResponse.ok) {
        const errorData = await apiResponse
          .json()
          .catch(() => ({ detail: "Error desconocido" }));
        throw new Error(errorData.detail || `Error HTTP ${apiResponse.status}`);
      }
      result = await apiResponse.json();
    } catch (err) {
      console.error("💥 Error en OCR:", err);
      error = err.message;
    } finally {
      isLoading = false;
    }
  }

  function toggleDropdown() {
    isOpen = !isOpen;
  }

  // ✅ Helper mejorado para mostrar datos
  function display(key, fallback = "No detectado") {
    if (!result) return fallback;
    const valor = result[key];
    if (valor === null || valor === undefined || valor === "") return fallback;
    return valor;
  }

  // ✅ Formatear monto
  function formatMonto(monto) {
    if (monto === null || monto === undefined || monto === "No detectado") {
      return "No detectado";
    }
    return Number(monto).toFixed(2);
  }

  // ✅ Color del badge según estado
  function getBadgeColor() {
    if (advertenciasExito)
      return "bg-green-500/20 text-green-400 border-green-500/30";
    if (advertenciasErrores.length > 0)
      return "bg-red-500/20 text-red-400 border-red-500/30";
    if (advertenciasWarnings.length > 0)
      return "bg-yellow-500/20 text-yellow-400 border-yellow-500/30";
    return "bg-blue-500/20 text-blue-400 border-blue-500/30";
  }

  function getBadgeText() {
    if (advertenciasExito) return "✅ Válido";
    if (advertenciasErrores.length > 0) return "❌ Con errores";
    if (advertenciasWarnings.length > 0) return "⚠️ Con advertencias";
    return "ℹ️ Procesado";
  }
</script>

<div
  class="w-full bg-white/10 rounded-lg backdrop-blur-sm px-3 py-2 flex flex-col items-center shadow-sm relative"
>
  <!-- Error de conexión -->
  {#if error}
    <div
      class="bg-red-500/20 border border-red-500/30 rounded-lg p-2 w-full text-xs text-red-400 mb-2 flex items-center gap-2"
    >
      <svg
        class="w-4 h-4 flex-shrink-0"
        fill="none"
        stroke="currentColor"
        viewBox="0 0 24 24"
      >
        <path
          stroke-linecap="round"
          stroke-linejoin="round"
          stroke-width="2"
          d="M12 8v4m0 4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"
        />
      </svg>
      <span>{error}</span>
    </div>
  {/if}

  <!-- Badge de estado -->
  {#if result && !isLoading}
    <div class="w-full mb-2">
      <div
        class="inline-flex items-center gap-2 px-3 py-1 rounded-full border text-xs font-medium {getBadgeColor()}"
      >
        {getBadgeText()}
        {#if advertenciasCount > 0}
          <span class="bg-white/20 px-2 py-0.5 rounded-full text-xs">
            {advertenciasCount}
          </span>
        {/if}
      </div>
    </div>
  {/if}

  <!-- Lista de errores críticos -->
  {#if advertenciasErrores.length > 0}
    <div
      class="w-full text-xs bg-red-500/20 border border-red-500/30 text-red-400 rounded-lg mb-2 py-2 px-2 flex flex-col gap-1.5"
    >
      <div class="font-semibold flex items-center gap-1">
        <svg
          class="w-4 h-4"
          fill="none"
          stroke="currentColor"
          viewBox="0 0 24 24"
        >
          <path
            stroke-linecap="round"
            stroke-linejoin="round"
            stroke-width="2"
            d="M12 9v2m0 4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"
          />
        </svg>
        Errores detectados ({advertenciasErrores.length}):
      </div>
      {#each advertenciasErrores as err}
        <div class="flex items-start gap-1 pl-2">
          <span class="text-red-500 font-bold">•</span>
          <span class="flex-1">{err}</span>
        </div>
      {/each}
    </div>
  {/if}

  <!-- Lista de advertencias -->
  {#if advertenciasWarnings.length > 0}
    <div
      class="w-full text-xs bg-yellow-500/20 border border-yellow-500/30 text-yellow-400 rounded-lg mb-2 py-2 px-2 flex flex-col gap-1.5"
    >
      <div class="font-semibold flex items-center gap-1">
        <svg
          class="w-4 h-4"
          fill="none"
          stroke="currentColor"
          viewBox="0 0 24 24"
        >
          <path
            stroke-linecap="round"
            stroke-linejoin="round"
            stroke-width="2"
            d="M12 9v2m0 4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"
          />
        </svg>
        Advertencias ({advertenciasWarnings.length}):
      </div>
      {#each advertenciasWarnings as adv}
        <div class="flex items-start gap-1 pl-2">
          <span class="text-yellow-500 font-bold">•</span>
          <span class="flex-1">{adv}</span>
        </div>
      {/each}
    </div>
  {/if}

  <!-- Botón para ver detalles -->
  <button
    on:click={toggleDropdown}
    class="bg-white/75 text-black hover:text-white/75 transition-all duration-200 text-xs rounded-lg hover:bg-black px-4 py-1.5 mt-1 flex items-center gap-2 select-none cursor-pointer disabled:opacity-50 disabled:cursor-not-allowed"
    disabled={isLoading || !result}
    aria-expanded={isOpen}
    aria-controls="dropdown-ocr-info"
    type="button"
  >
    <span>Ver datos del comprobante</span>
    <svg
      class="dropdown-arrow w-4 h-4 {isOpen ? 'open' : ''}"
      fill="none"
      stroke="currentColor"
      viewBox="0 0 24 24"
    >
      <path
        stroke-linecap="round"
        stroke-linejoin="round"
        stroke-width="2"
        d="M19 9l-7 7-7-7"
      />
    </svg>
  </button>

  <!-- Dropdown con información detallada -->
  {#if result && !isLoading}
    <div
      id="dropdown-ocr-info"
      class="dropdown-content {isOpen
        ? ''
        : 'closed'} w-full bg-white brightness-90 border rounded-lg mt-2 px-3 py-2.5 shadow-md flex flex-col gap-2 text-xs"
      style="position: absolute; bottom: 110%; left: 0; z-index: 50; min-width: 300px; max-height: 450px; overflow-y: auto;"
      aria-hidden={!isOpen}
    >
      <!-- Tipo de transacción -->
      {#if result.tipo_transaccion}
        <div class="bg-blue-50 rounded px-2 py-1 mb-1">
          <span class="font-semibold text-blue-700 text-sm">
            {result.tipo_transaccion}
          </span>
        </div>
      {/if}

      <!-- Datos principales -->
      <div class="space-y-1.5">
        <!-- Monto (destacado) -->
        <div
          class="flex justify-between items-center bg-green-50 rounded px-2 py-1.5 border border-green-200"
        >
          <span class="font-semibold text-green-800">Monto:</span>
          <span class="text-right text-green-900 font-bold text-sm">
            S/ {formatMonto(display("monto"))}
          </span>
        </div>

        <!-- Código de operación -->
        <div
          class="flex justify-between items-center border-b border-black/10 pb-1"
        >
          <span class="font-semibold text-black/75">Nro. Op.:</span>
          <span class="text-right text-black/75 font-mono text-xs">
            {display("codigo_operacion")}
          </span>
        </div>

        <!-- Receptor -->
        <div
          class="flex justify-between items-center border-b border-black/10 pb-1"
        >
          <span class="font-semibold text-black/75">Receptor:</span>
          <span
            class="text-right text-black/75 max-w-[160px] truncate"
            title={display("receptor")}
          >
            {display("receptor")}
          </span>
        </div>

        <!-- Teléfono -->
        {#if result.telefono}
          <div
            class="flex justify-between items-center border-b border-black/10 pb-1"
          >
            <span class="font-semibold text-black/75">Teléfono:</span>
            <span class="text-right text-black/75 font-mono text-xs">
              {display("telefono")}
            </span>
          </div>
        {/if}

        <!-- Fecha y hora -->
        <div
          class="flex justify-between items-center border-b border-black/10 pb-1"
        >
          <span class="font-semibold text-black/75">Fecha:</span>
          <span class="text-right text-black/75">
            {display("fecha")}
          </span>
        </div>

        <div
          class="flex justify-between items-center border-b border-black/10 pb-1"
        >
          <span class="font-semibold text-black/75">Hora:</span>
          <span class="text-right text-black/75">
            {display("hora")}
          </span>
        </div>

        <!-- Destino -->
        <div
          class="flex justify-between items-center border-b border-black/10 pb-1"
        >
          <span class="font-semibold text-black/75">Destino:</span>
          <span class="text-right text-black/75 font-medium">
            {display("destino")}
          </span>
        </div>

        <!-- Comisión -->
        {#if result.comision}
          <div class="flex justify-between items-center">
            <span class="font-semibold text-black/75">Comisión:</span>
            <span
              class="text-right text-black/75 {result.comision === 'GRATIS'
                ? 'text-green-700 font-bold'
                : ''}"
            >
              {display("comision")}
            </span>
          </div>
        {/if}
      </div>

      <!-- Comentario (si existe) -->
      {#if result.comentario}
        <div class="mt-2 pt-2 border-t border-black/10">
          <span class="font-semibold text-black/75 block mb-1">Comentario:</span
          >
          <span
            class="text-black/60 text-xs italic bg-gray-50 rounded px-2 py-1 block"
          >
            "{result.comentario}"
          </span>
        </div>
      {/if}
    </div>
  {/if}

  <!-- Estado de carga -->
  {#if isLoading}
    <div class="w-full text-center py-6">
      <div
        class="w-10 h-10 border-3 border-purple-600 border-t-transparent rounded-full animate-spin mx-auto mb-3"
      ></div>
      <h3 class="text-white/75 text-sm font-medium mb-1">
        Procesando comprobante...
      </h3>
      <p class="text-white/50 text-xs">Extrayendo datos con OCR</p>
    </div>
  {/if}

  <!-- Estado sin imagen -->
  {#if !hasImage && !isLoading}
    <div class="w-full text-center py-6">
      <svg
        class="w-14 h-14 text-gray-400 mx-auto mb-3"
        fill="none"
        stroke="currentColor"
        viewBox="0 0 24 24"
      >
        <path
          stroke-linecap="round"
          stroke-linejoin="round"
          stroke-width="2"
          d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z"
        />
      </svg>
      <h3 class="text-white/75 text-sm font-medium mb-1">Sin comprobante</h3>
      <p class="text-white/50 text-xs">Sube una imagen para validar</p>
    </div>
  {/if}
</div>

<style>
  .dropdown-arrow {
    transition: transform 0.25s;
  }
  .dropdown-arrow.open {
    transform: rotate(-180deg);
  }
  .dropdown-content {
    transition:
      opacity 0.2s,
      transform 0.2s;
    opacity: 1;
    transform: translateY(0);
    pointer-events: auto;
  }
  .dropdown-content.closed {
    opacity: 0;
    transform: translateY(20px);
    pointer-events: none;
  }
</style>
