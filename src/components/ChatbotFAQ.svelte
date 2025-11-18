<script>
    import { onMount } from "svelte";

    let isOpen = false;
    let messages = [];
    let showQuestions = true;

    const faqs = [
        {
            id: 1,
            question: "¿Qué tipos de imágenes puedo verificar?",
            answer: "Puedes verificar capturas de pantalla de transacciones Plin en formato JPG, PNG o WEBP. La imagen debe mostrar claramente los detalles de la transacción.",
        },
        {
            id: 2,
            question: "¿Cómo funciona la verificación?",
            answer: "Nuestro sistema utiliza filtros inteligentes y comparación con plantillas para analizar la autenticidad de la imagen. Verificamos elementos como el formato, colores, tipografía y estructura del comprobante Plin.",
        },
        {
            id: 3,
            question: "¿Qué hago si mi imagen es rechazada?",
            answer: "Si tu imagen es rechazada, asegúrate de que sea una captura de pantalla original sin ediciones, que tenga buena resolución y que muestre todos los elementos de la transacción Plin claramente.",
        },
        {
            id: 4,
            question: "¿Es seguro subir mis imágenes?",
            answer: "Sí, las imágenes se procesan de forma segura y no se almacenan permanentemente en nuestros servidores. Solo se utilizan para la verificación inmediata.",
        },
        {
            id: 5,
            question: "¿Cuánto tiempo tarda la verificación?",
            answer: "La verificación es casi instantánea. Nuestro sistema procesa la imagen en pocos segundos y te muestra los resultados inmediatamente.",
        },
        {
            id: 6,
            question: "¿Qué filtros se aplican a la imagen?",
            answer: "Aplicamos múltiples filtros como análisis de metadatos, detección de manipulación, verificación de elementos visuales del diseño Plin y comparación con plantillas oficiales.",
        },
    ];

    function toggleChat() {
        isOpen = !isOpen;
        if (isOpen && messages.length === 0) {
            messages = [
                {
                    type: "bot",
                    text: "¡Hola! 👋 Soy tu asistente virtual. ¿En qué puedo ayudarte hoy?",
                    timestamp: new Date(),
                },
            ];
        }
    }

    function selectQuestion(faq) {
        messages = [
            ...messages,
            {
                type: "user",
                text: faq.question,
                timestamp: new Date(),
            },
        ];

        showQuestions = false;

        setTimeout(() => {
            messages = [
                ...messages,
                {
                    type: "bot",
                    text: faq.answer,
                    timestamp: new Date(),
                },
            ];

            setTimeout(() => {
                showQuestions = true;
            }, 500);
        }, 600);
    }

    function resetChat() {
        messages = [
            {
                type: "bot",
                text: "¡Hola! 👋 Soy tu asistente virtual. ¿En qué puedo ayudarte hoy?",
                timestamp: new Date(),
            },
        ];
        showQuestions = true;
    }

    $: if (messages.length > 0) {
        setTimeout(() => {
            const container = document.getElementById("messages-container");
            if (container) {
                container.scrollTop = container.scrollHeight;
            }
        }, 100);
    }
</script>

<!-- Botón flotante -->
<button
    on:click={toggleChat}
    class="fixed bottom-6 right-6 z-50 w-14 h-14 bg-gradient-to-r from-[#0581F6] to-[#16DEC8] rounded-full shadow-lg hover:shadow-xl transition-all duration-300 flex items-center justify-center group hover:scale-110 cursor-pointer"
    aria-label="Abrir chat de ayuda"
>
    {#if isOpen}
        <svg
            class="w-6 h-6 text-white"
            fill="none"
            stroke="currentColor"
            viewBox="0 0 24 24"
        >
            <path
                stroke-linecap="round"
                stroke-linejoin="round"
                stroke-width="2"
                d="M6 18L18 6M6 6l12 12"
            />
        </svg>
    {:else}
        <svg
            class="w-6 h-6 text-white"
            fill="none"
            stroke="currentColor"
            viewBox="0 0 24 24"
        >
            <path
                stroke-linecap="round"
                stroke-linejoin="round"
                stroke-width="2"
                d="M8 10h.01M12 10h.01M16 10h.01M9 16H5a2 2 0 01-2-2V6a2 2 0 012-2h14a2 2 0 012 2v8a2 2 0 01-2 2h-5l-5 5v-5z"
            />
        </svg>
    {/if}
    <span
        class="absolute -top-1 -right-1 w-3 h-3 bg-red-500 rounded-full animate-pulse"
    ></span>
</button>

<!-- Ventana del chat -->
{#if isOpen}
    <div
        class="fixed bottom-24 right-6 z-50 w-96 h-[32rem] bg-gray-900/95 backdrop-blur-lg rounded-2xl shadow-2xl border border-white/10 flex flex-col overflow-hidden animate-in fade-in slide-in-from-bottom-4 duration-300"
    >
        <!-- Header del chat -->
        <div
            class="bg-gradient-to-r from-[#0581F6] to-[#16DEC8] p-4 flex items-center justify-between"
        >
            <div class="flex items-center gap-3">
                <div
                    class="w-10 h-10 bg-white/20 rounded-full flex items-center justify-center"
                >
                    <svg
                        class="w-6 h-6 text-white"
                        fill="none"
                        stroke="currentColor"
                        viewBox="0 0 24 24"
                    >
                        <path
                            stroke-linecap="round"
                            stroke-linejoin="round"
                            stroke-width="2"
                            d="M18.364 5.636l-3.536 3.536m0 5.656l3.536 3.536M9.172 9.172L5.636 5.636m3.536 9.192l-3.536 3.536M21 12a9 9 0 11-18 0 9 9 0 0118 0zm-5 0a4 4 0 11-8 0 4 4 0 018 0z"
                        />
                    </svg>
                </div>
                <div>
                    <h3 class="font-semibold text-white">Asistente Virtual</h3>
                    <p class="text-xs text-white/80">Preguntas frecuentes</p>
                </div>
            </div>
            <button
                on:click={resetChat}
                class="text-white/80 hover:text-white transition-colors"
                title="Reiniciar chat"
                aria-label="Reiniciar chat"
            >
                <svg
                    class="w-5 h-5"
                    fill="none"
                    stroke="currentColor"
                    viewBox="0 0 24 24"
                >
                    <path
                        stroke-linecap="round"
                        stroke-linejoin="round"
                        stroke-width="2"
                        d="M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15"
                    />
                </svg>
            </button>
        </div>

        <!-- Mensajes -->
        <div
            id="messages-container"
            class="flex-1 overflow-y-auto p-4 space-y-4 scroll-smooth"
        >
            {#each messages as message}
                <div
                    class="flex {message.type === 'user'
                        ? 'justify-end'
                        : 'justify-start'}"
                >
                    <div
                        class="max-w-[80%] {message.type === 'user'
                            ? 'bg-gradient-to-r from-[#0581F6] to-[#16DEC8]'
                            : 'bg-gray-800'} rounded-2xl px-4 py-2.5 shadow-lg"
                    >
                        <p class="text-sm text-white">{message.text}</p>
                    </div>
                </div>
            {/each}

            {#if showQuestions && messages.length > 0}
                <div class="space-y-2 pt-2">
                    <p class="text-xs text-white/60 font-medium px-1">
                        Preguntas frecuentes:
                    </p>
                    {#each faqs as faq}
                        <button
                            on:click={() => selectQuestion(faq)}
                            class="w-full text-left bg-gray-800/50 hover:bg-gray-800 border border-white/5 hover:border-[#0581F6]/50 rounded-xl px-4 py-3 transition-all duration-200 group"
                        >
                            <p
                                class="text-sm text-white/90 group-hover:text-white"
                            >
                                {faq.question}
                            </p>
                        </button>
                    {/each}
                </div>
            {/if}
        </div>
    </div>
{/if}

<style>
    @keyframes fade-in {
        from {
            opacity: 0;
        }
        to {
            opacity: 1;
        }
    }

    @keyframes slide-in-from-bottom-4 {
        from {
            transform: translateY(1rem);
        }
        to {
            transform: translateY(0);
        }
    }

    .animate-in {
        animation:
            fade-in 0.3s ease-out,
            slide-in-from-bottom-4 0.3s ease-out;
    }
</style>
