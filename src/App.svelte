<script>
    import { onMount } from 'svelte';
    import { quintOut } from 'svelte/easing';
    import { fade } from 'svelte/transition';
    import * as d3 from "d3";

    // --- Variables de estado generales ---
    let sectionActive = 'galaxy-intro'; // Rastrea la sección visible para efectos
    let showLegend = false; // Controla la visibilidad de la leyenda
    let isTopProfessorsMode = false; // Controla la visibilidad del overlay del Top 4
    let selectedProfessor = null; // Para el modal de detalles del profesor
    let filter = "all"; // Filtro de "Recomendados", "No Recomendados", "Todos"
    let filterSubject = "all"; // Filtro por materia

    // --- Variables para la sección de avatares ---
    let showAvatarCreator = false; // Controla la visibilidad de la sección de creación de avatares
    let selectedAvatarProfessor = null; // Profesor seleccionado para ver/personalizar su avatar
    let currentAvatarIndex = 0; // Para la navegación de avatares
    let showCreatorModal = false;


    // --- Datos de Profesores ---
    let stars = [
        // LIKE 10: (Usar 29.5 para todas)
        { id: 1, name: "Carlos Araujo", subject: "Visualización Datos", year: 2025, like: 10, difficulty: 3, recommended: "Sí", mood: "Divertido", x: 25, y: 29.5, gender: "male",
            description: "Un diseñador único que se enfoca en cómo transformar datos complejos en historias visuales impactantes. Sus clases son muy prácticas y te enseñan a usar herramientas modernas, siempre dispuesto a responder consultas.",
            phrase: "Los datos hablan, pero la visualización les da voz. Mi profesor favorito hasta al momento" },
        { id: 2, name: "Nicolas Allo Gomez", subject: "Álgebra Lineal", year: 2023, like: 10, difficulty: 5, recommended: "Sí", mood: "Divertido", x: 40, y: 29.5, gender: "male",
            description: "Un experto en las bases del álgebra lineal, muy alegre en sus explicaciones y atento para ayudar a sus alumnos.",
            phrase: "Sus risas le dan vida a las clases tan díficiles. No hay nadie como Nico, es especial" },
        { id: 5, name: "Gervasio Perez", subject: "Tecnología Digital III", year: 2025, like: 10, difficulty: 7, recommended: "Sí", mood: "Interesante", x: 55, y: 29.5, gender: "male",
            description: "Clásico, pero muy eficaz en sus explicaciones y tiene muy buena onda a responderte.",
            phrase: "El futuro no se predice, se construye con tecnología." },
        { id: 13, name: "Victoria Venuti", subject: "Matemática II", year: 2023, like: 10, difficulty: 4, recommended: "Sí", mood: "Divertido", x: 70, y: 29.5, gender: "female",
            description: "Enseña los principios de cálculo con una claridad excepcional. Sus ejercicios te preparan para cualquier desafío cuantitativo.",
            phrase: "Las matemáticas son el lenguaje del universo." },

        // LIKE 9: (Usar 38.5 para todas)
        { id: 19, name: "Tomás Curzio", subject: "Visualización Datos", year: 2025, like: 9, difficulty: 2, recommended: "Sí", mood: "Divertido", x: 25, y: 38.5, gender: "male",
            description: "Con un enfoque muy práctico, este profesor te enseña a crear codificaciones que no solo informan, sino que también inspiran. Ideal para proyectos reales.",
            phrase: "Transformando números en narrativas visuales." },
        { id: 4, name: "Felipe Tappata", subject: "Microeconomía", year: 2024, like: 9, difficulty: 4, recommended: "Sí", mood: "Interesante", x: 35, y: 38.5, gender: "male",
            description: "Explora las decisiones económicas a nivel individual y empresarial, con ejemplos claros y debates enriquecedores que te hacen pensar.",
            phrase: "Con mucha onda, pero a su vez con seriedad te hace entender" },
        { id: 12, name: "Tomas Bustos", subject: "Economía", year: 2023, like: 9, difficulty: 5, recommended: "Sí", mood: "Divertido", x: 50, y: 38.5, gender: "male",
            description: "Un profesor que hace que la economía cobre vida, mostrando cómo afecta nuestro día a día y las políticas globales, sus gráficos en Ipad son únicos.",
            phrase: "La economía es la ciencia de la vida cotidiana." },
        { id: 15, name: "Romina Botzman", subject: "Matemática", year: 2024, like: 9, difficulty: 3, recommended: "Sí", mood: "Divertido", x: 65, y: 38.5, gender: "female",
            description: "Hace que los números dejen de ser un misterio. Su metodología es clara y su paciencia infinita, ideal para entender conceptos complejos.",
            phrase: "Descubriendo la belleza en cada ecuación." },
        { id: 17, name: "Emilia Ghelfi", subject: "Expresión Oral y Escrita", year: 2025, like: 9, difficulty: 2, recommended: "Sí", mood: "Divertido", x: 80, y: 38.5, gender: "female",
            description: "Te enseña a comunicar tus ideas con claridad y persuasión, tanto al hablar como al escribir. Indispensable para cualquier profesión.",
            phrase: "Las palabras son puentes hacia el entendimiento." },

        // LIKE 8: (Usar 47.5 para todas)
        { id: 3, name: "Francisco Ciocchini", subject: "Microeconomía", year: 2024, like: 8, difficulty: 5, recommended: "Sí", mood: "Divertido", x: 25, y: 47.5, gender: "male",
            description: "Conocedor profundo de la microeconomía, sus clases son estructuradas, pero con simpatía, te preparan para analizar mercados y comportamientos de consumidores.",
            phrase: "El mercado es un diálogo de decisiones." },
        { id: 20, name: "Juana Copello", subject: "Visualización Datos", year: 2025, like: 8, difficulty: 2, recommended: "Sí", mood: "Interesante", x: 40, y: 47.5, gender: "female",
            description: "Experta en narración de datos, te guía para que tus gráficos cuenten historias cautivadoras y fáciles de entender.",
            phrase: "Una imagen vale más que mil datos crudos." },
        { id: 18, name: "Michelle Baros", subject: "Comprensión de Textos", year: 2024, like: 8, difficulty: 4, recommended: "Sí", mood: "Interesante", x: 55, y: 47.5, gender: "female",
            description: "Analiza los secretos de la lectura efectiva y el análisis crítico, una habilidad esencial en cualquier campo de estudio.",
            phrase: "Te va pedir que hagas varias veces el trabajo, pero al final sacas A." },
        { id: 9, name: "Francisco Corigliano", subject: "Historia Occidental", year: 2022, like: 8, difficulty: 1, recommended: "Sí", mood: "Interesante", x: 70, y: 47.5, gender: "male",
            description: "Un viaje fascinante por la historia de Occidente, conectando el pasado con el presente de una manera que te hará reflexionar.",
            phrase: "El pasado es la brújula del futuro." },
           { id: 22, name: "Joaquín Navajas", subject: "Neurociencias", year: 2023, like: 8, difficulty: 6, recommended: "Sí", mood: "Interesante", x: 80, y: 47.5, gender: "male", 
        description: "Un especialista en neurociencias y psicología experimental, sus clases ofrecen una perspectiva profunda sobre el funcionamiento de la mente y el comportamiento humano, con casos de estudio muy interesantes.",
        phrase: "La mente es un vasto universo de posibilidades." },

        // LIKE 7: (Usar 56.5 para todas)
        { id: 16, name: "Javier Fuentes", subject: "Contabilidad", year: 2023, like: 7, difficulty: 5, recommended: "Sí", mood: "Interesante", x: 30, y: 56.5, gender: "male",
            description: "Domina el arte de los números financieros y te enseña a interpretarlos para tomar decisiones informadas en cualquier negocio.",
            phrase: "Cada número cuenta una historia financiera." },
        { id: 6, name: "Juan Edi", subject: "Tecnología Digital 3", year: 2025, like: 7, difficulty: 5, recommended: "Sí", mood: "Interesante", x: 50, y: 56.5, gender: "male",
            description: " Siempre hay un buen chiste, profesor cómico.",
            phrase: "Te deriva al foro todo el tiempo, pero lo banco." },
        { id: 11, name: "David Laviosa", subject: "Gestión Proyectos", year: 2025, like: 7, difficulty: 2, recommended: "Sí", mood: "Interesante", x: 70, y: 56.5, gender: "male",
            description: "Trabaja en PedidosYA, me enseñándote metodologías para armar una buena empresa, desde la planificación hasta la ejecución.",
            phrase: "La clave del éxito reside en una gestión impecable." },
            { id: 23, name: "Fabrizio Rodriguez", subject: "Microeconomía", year: 2024, like: 7, difficulty: 7, recommended: "Sí", mood: "Interesante", x: 80, y: 56.5, gender: "male",
            description: "Es un docente amable y dedicado, que genera un ambiente interesante donde los estudiantes se sienten apoyados para aprender y crecer.", phrase: "Un uruguayo en Ditella"},

        // LIKE 6: (Usar 65 para todas)
        { id: 21, name: "Fernanda Milman", subject: "Contabilidad", year: 2024, like: 6, difficulty: 6, recommended: "No", mood: "Interesante", x: 45, y: 65, gender: "female",
            description: "Si bien puede ser un desafío, sus clases de contabilidad te brindan una base sólida para entender las finanzas corporativas.",
            phrase: "Los balances son el pulso de la empresa." },

        // LIKE < 6 (Usar 74.5 para todas)
        { id: 8, name: "David Gonzalez", subject: "TD2", year: 2023, like: 5, difficulty: 9, recommended: "No", mood: "Aburrido", x: 25, y: 74.5, gender: "male",
            description: "Un profesor que te desafiará con la complejidad de la Tecnología Digital 2, requiriendo mucha dedicación para dominarla.",
            phrase: "La tecnología exige disciplina y estudio constante." },
        { id: 14, name: "Martín Vacas", subject: "Matemática", year: 2023, like: 3, difficulty: 4, recommended: "No", mood: "Aburrido", x: 40, y: 74.5, gender: "male",
            description: "Sus clases de matemática se entienden, pero no tiene onda para explicar.",
            phrase: "Siempre tomando mate." },
        { id: 7, name: "Pablo Dobrusin", subject: "TD2", year: 2023, like: 2, difficulty: 8, recommended: "No", mood: "Aburrido", x: 60, y: 74.5, gender: "male",
            description: "La materia con este profesor puede ser un verdadero reto. Prepárate para un aprendizaje exclusivamente autónomo.",
            phrase: "El camino del conocimiento es a menudo solitario." },
        { id: 10, name: "Gabriela Spinetto", subject: "Diseño", year: 2022, like: 2, difficulty: 6, recommended: "No", mood: "Aburrido", x: 80, y: 74.5, gender: "female",
            description: "Sus clases de diseño son menos dinámicas de lo esperado, lo que puede dificultar la inspiración creativa.",
            phrase: "La creatividad necesita chispa, no solo reglas." },
    ];

    // Define el orden específico para el Top 4 visual
    const top4Order = [
        "Carlos Araujo",
        "Gervasio Perez",
        "Nicolas Allo Gomez",
        "Victoria Venuti"
    ];

    // Mapeo de puntajes a la posición Y central de su fila
    const scoreYPositions = {
        10: 29.5, // Promedio de 28-31
        9: 38.5,  // Promedio de 37-40
        8: 47.5,  // Promedio de 46-49
        7: 56.5,  // Promedio de 55-58
        6: 65,    // Fila para puntaje 6
        "<6": 74.5 // Promedio de 73-76
    };

    // Datos para la sección de "muñequitos"
    // Usaremos los mismos profesores de 'stars' para los avatares
    const professorAvatars = stars.map(p => ({
        id: p.id,
        name: p.name,
        subject: p.subject,
        mood: p.mood,
        gender: p.gender,
        color: starColor(p), // Reutilizo la función starColor para darles un color base
        description: p.description,
        phrase: p.phrase 
    }));

    // --- Funciones y lógica principal ---
   // Lógica para las estrellas mostradas en la galaxia (filtrado)
$: displayedStars = (() => {
    if (isTopProfessorsMode) {
        return getRankedStars();
    } else {
        return stars.filter(s => {
            let passesLikeFilter = true;
            if (filter === "liked") {
                passesLikeFilter = s.like >= 7;
            } else if (filter === "disliked") {
                passesLikeFilter = s.like < 7;
            }

            let passesSubjectFilter = true;
            if (filterSubject !== "all") {
                passesSubjectFilter = s.subject === filterSubject;
            }

            return passesLikeFilter && passesSubjectFilter;
        });
    }
})();

    // Función para manejar el scroll snap y determinar la sección activa
    function handleScroll(event) {
        const sections = document.querySelectorAll('.scroll-section');
        const scrollY = event.target.scrollTop;
        const viewportHeight = window.innerHeight;

        sections.forEach(section => {
            const sectionTop = section.offsetTop;
            const sectionBottom = sectionTop + section.offsetHeight;

            // Determinar la sección activa cuando la mitad de la sección está en el viewport
            if (scrollY >= sectionTop - viewportHeight / 2 && scrollY < sectionBottom - viewportHeight / 2) {
                sectionActive = section.id;
            }
        });
    }

    // Funciones para abrir y cerrar el modal de detalles del profesor
    function openProfessorDetails(professor) {
        isTopProfessorsMode = false; // Asegura que el top professors esté oculto
        selectedProfessor = professor; // Esto es para el modal de detalles de la ESTRELLA
    }

    function closeProfessorDetails() {
        selectedProfessor = null;
    }

    // La misma idea, pero para los muñecos
    function openAvatarDetails(professor) {
        selectedAvatarProfessor = professor; 
    }

    function closeAvatarDetails() {
        selectedAvatarProfessor = null;
    }

    // Filtro de estrellas (controla los botones "Todos", "Recomendados", "No Recomendados")
    function setFilter(value) {
        filter = value;
        isTopProfessorsMode = false; // Desactiva el modo Top 4 al cambiar de filtro
    }

    // Filtro por materia 
    function setFilterBySubject(subject) {
        filterSubject = subject;
        filter = "all";
        isTopProfessorsMode = false;
    }

    // Obtiene el color de la estrella según el "mood" del profesor
    function starColor(star) {
        switch (star.mood.toLowerCase()) {
            case "divertido": return "#00ffcc";
            case "interesante": return "#ffd700";
            case "aburrido": return "#ff69b4";
            default:
                if (star.recommended === "No") return "#666";
                return "#ccc";
        }
    }

    // Ajusta el tamaño de la estrella basado en su "like" o "rank"
    function starSize(like, rank) {
        if (rank !== undefined) {
            // Tamaños para el Top 4 rankeado
            if (rank === 1) return 85;
            if (rank === 2) return 70;
            if (rank === 3) return 60;
            if (rank === 4) return 50;
        }

        // Define un rango de tamaños base y una proporción de incremento
        const baseSizeRecommended = 30; // Tamaño mínimo para estrellas recomendadas (>=7)
        const baseSizeNotRecommended = 20; // Tamaño mínimo para no recomendadas (<7)
        const sizeMultiplier = 12; // Multiplicador para la progresión

        if (like >= 7) {
            // Para puntajes 7, 8, 9, 10
            return baseSizeRecommended + (sizeMultiplier * (like - 7));
        } else {
            // Para puntajes < 7 (los "No Recomendados"), incluyendo el 6
            // Queremos que 6 sea visible pero claramente más pequeño que 7
            // Y los puntajes más bajos tengan un tamaño mínimo aceptable
            return baseSizeNotRecommended + (sizeMultiplier * (like - 2) * 0.5);
        }
    }

    // Determina si una estrella debe tener el "glow" de favorito
    function isFavorite(star) {
        return star.like >= 9; 
    }

 // Prepara los datos para el overlay del Top 4 (estrellas ordenadas y posicionadas)
function getRankedStars() {
    const ranked = [];
    // Ajusto la posición Y para que estén más arriba en el centro de la pantalla
    // y el espaciado X para que se distribuyan bien en formato de ranking (escalonado).

    const base_y = 35; // Posición Y base más arriba
    const initial_x = 25; // Posición X inicial para la primera estrella
    const spacing_x = 18; // Espaciado horizontal entre estrellas
    const y_step = 10; // Incremento de Y para el efecto de ranking (escalonado)

    top4Order.forEach((name, index) => {
        const professor = stars.find(s => s.name === name);
        if (professor) {
            ranked.push({
                ...professor,
                // Posición X para distribuir horizontalmente
                x: initial_x + (index * spacing_x) + (index * 5), 
                // Posición Y escalonada para el efecto de ranking
                y: base_y + (index * y_step),
                rank: index + 1 // Añado el rank para el tamaño
            });
        }
    });
    return ranked;
}

    // Obtiene los datos del Top 4 para la lista dentro del overlay
    function getTopProfessorsForOverlay() {
        const topProfessors = [];
        top4Order.forEach(name => {
            const professor = stars.find(s => s.name === name);
            if (professor) {
                topProfessors.push(professor);
            }
        });
        return topProfessors;
    }

    // Obtiene el SVG de la medalla para el Top 4 (por rank)
    function getMedalSvg(rank) {
        switch (rank) {
            case 1: return '<svg width="24" height="24" viewBox="0 0 24 24" fill="#FFD700"><path d="M12 2L9.19 8.62L2 9.24L7.46 13.97L5.82 21L12 17.27L18.18 21L16.54 13.97L22 9.24L14.81 8.62L12 2Z"/></svg>';
            case 2: return '<svg width="24" height="24" viewBox="0 0 24 24" fill="#C0C0C0"><path d="M12 2L9.19 8.62L2 9.24L7.46 13.97L5.82 21L12 17.27L18.18 21L16.54 13.97L22 9.24L14.81 8.62L12 2Z"/></svg>';
            case 3: return '<svg width="24" height="24" viewBox="0 0 24 24" fill="#CD7F32"><path d="M12 2L9.19 8.62L2 9.24L7.46 13.97L5.82 21L12 17.27L18.18 21L16.54 13.97L22 9.24L14.81 8.62L12 2Z"/></svg>';
            case 4: return '<svg width="24" height="24" viewBox="0 0 24 24" fill="#E0E0E0"><path d="M12 2L9.19 8.62L2 9.24L7.46 13.97L5.82 21L12 17.27L18.18 21L16.54 13.97L22 9.24L14.81 8.62L12 2Z"/></svg>';
            default: return '';
        }
    }

    // Genera el HTML de la lista de profesores para el overlay
    function showTopProfessorsContent() {
        const topProfessors = getTopProfessorsForOverlay();
        let htmlContent = '';
        topProfessors.forEach((professor, index) => {
            htmlContent += `
                <li>
                    <span class="top-star-medal">${getMedalSvg(index + 1)}</span>
                    <span>${professor.name} - ${professor.subject}</span>
                </li>
            `;
        });
        return htmlContent;
    }
// Genera el SVG del avatar basado en la materia y género
function generateProfessorAvatarSvg(professor) {

let skinColor = '#FAD9C5';
let hair = '';
let bodyBaseColor = professor.color; // Asume que 'professor.color' ya está definido o se manejará en el default
let accessory = '';
let features = '';


switch (professor.subject.toLowerCase()) {
    case "visualización datos":
    case "diseño":
    bodyBaseColor = '#8D40C4';

    // Accesorio visual base
    accessory = `
        <circle cx="50" cy="40" r="25" fill="none" stroke="#fff" stroke-width="2"/>
        <text x="50" y="75" font-family="Arial" font-size="10" fill="#fff" text-anchor="middle">📊</text>
    `;

    if (professor.gender === "female") {
        // Pelo largo, prolijo y con ondas suaves ☁️
        hair = `
            <path d="M30 20 C 20 30, 15 50, 25 65 C 35 80, 65 80, 75 65 C 85 50, 80 30, 70 20 Q 50 5, 30 20 Z" fill="#1C1C1C"/>
        `;

        // Anteojos ovalados elegantes 🕶️✨
        accessory += `
            <ellipse cx="35" cy="40" rx="6" ry="4" fill="black"/>
            <ellipse cx="65" cy="40" rx="6" ry="4" fill="black"/>
            <line x1="41" y1="40" x2="59" y2="40" stroke="black" stroke-width="2"/>
            <line x1="29" y1="40" x2="25" y2="38" stroke="black" stroke-width="2"/>
            <line x1="71" y1="40" x2="75" y2="38" stroke="black" stroke-width="2"/>
        `;
    } else {
        // Pelo corto con onda (para el varón)
        hair = `
            <path d="M25 25 C 30 10, 70 10, 75 25 C 80 40, 80 60, 70 65 C 60 70, 40 70, 30 65 C 20 60, 20 40, 25 25 Z" fill="#1C1C1C"/>
        `;

        // Anteojos rectangulares cancheros 😎
        accessory += `
            <rect x="30" y="35" width="15" height="10" fill="black" rx="2" ry="2"/>
            <rect x="55" y="35" width="15" height="10" fill="black" rx="2" ry="2"/>
            <line x1="45" y1="40" x2="55" y2="40" stroke="black" stroke-width="2"/>
            <line x1="30" y1="40" x2="25" y2="38" stroke="black" stroke-width="2"/>
            <line x1="70" y1="40" x2="75" y2="38" stroke="black" stroke-width="2"/>
        `;
    }

    break;



    case "tecnología digital iii":
    case "tecnología digital 3":
    case "td2":
        bodyBaseColor = '#2ECC71';
        accessory = `<rect x="30" y="35" width="40" height="15" rx="5" ry="5" fill="#333"/><circle cx="50" cy="42.5" r="3" fill="#00ffcc"/>`; // Anteojos
        features += `<path d="M 40 50 L 45 55 L 55 55 L 60 50" stroke="#333" stroke-width="2" fill="none"/>`;
        if (professor.gender === "female") {
                hair = `<path d="M25 25 C 10 20, 10 55, 25 60 C 40 65, 60 65, 75 60 C 90 55, 90 20, 75 25" fill="#A0522D"/>`;
        } else {
            hair = `<path d="M30 20 Q 50 5 70 20 C 75 30, 75 50, 70 60 Q 50 75 30 60 C 25 50, 25 30, 30 20 Z" fill="#4B382D"/>`;
        }
        break;
        case "matemática":
case "matemática ii":
case "álgebra lineal":
    bodyBaseColor = '#3498DB'; // Azul clásico matemático

    // Color especial para Nico Allo
    if (professor.name === "Nicolas Allo Gomez") {
        bodyBaseColor = '#F39C12'; // Dorado brillante
    }

    // Inicialización
    hair = "";
    accessory = "";

    // Estilos por género
    if (professor.gender === "female") {
        // Pelo con forma natural y cálida
        hair = `
            <path d="M25 20 C 10 25, 10 55, 25 65 Q 50 80, 75 65 C 90 55, 90 25, 75 20 Q 65 15, 50 15 Q 35 15, 25 20 Z" fill="#6E4F3A"/>
        `;

        // Lentes redondos, suaves y elegantes
        accessory = `
            <circle cx="36" cy="40" r="7" fill="none" stroke="#5D6D7E" stroke-width="2"/>
            <circle cx="64" cy="40" r="7" fill="none" stroke="#5D6D7E" stroke-width="2"/>
            <line x1="43" y1="40" x2="57" y2="40" stroke="#5D6D7E" stroke-width="1.5"/>
        `;
    } else {
        // Pelo masculino clásico
        hair = `
            <path d="M30 20 Q 50 8, 70 20 C 75 35, 75 50, 70 60 Q 50 72, 30 60 C 25 50, 25 35, 30 20 Z" fill="#3E2F1C"/>
        `;

        // Lentes rectangulares
        accessory = `
            <rect x="30" y="35" width="16" height="10" fill="none" stroke="black" stroke-width="2"/>
            <rect x="54" y="35" width="16" height="10" fill="none" stroke="black" stroke-width="2"/>
            <line x1="46" y1="40" x2="54" y2="40" stroke="black" stroke-width="2"/>
        `;
    }

    // 💥 Pelo rojo especial para Nico
    if (professor.name === "Nicolas Allo Gomez") {
        hair = `
            <path d="M30 20 Q 50 8, 70 20 C 75 35, 75 50, 70 60 Q 50 72, 30 60 C 25 50, 25 35, 30 20 Z" fill="#E74C3C"/>
        `;
    }

    break;

    case "contabilidad":
    bodyBaseColor = '#1ABC9C'; // Un verde agua profesional y fresco

    // Inicialización de elementos
    hair = "";
    accessory = "";

    if (professor.gender === "female") {
        // Pelo suelto liso con estilo práctico
        hair = `
            <path d="M20 25 C 15 15, 85 15, 80 25 C 85 40, 85 55, 75 65 C 65 70, 35 70, 25 65 C 15 55, 15 40, 20 25 Z" fill="#5D4037"/>
        `;

        // Lentes con marco de color (onda sofisticada)
        accessory = `
            <rect x="28" y="35" width="14" height="10" fill="none" stroke="#8E44AD" stroke-width="2"/>
            <rect x="58" y="35" width="14" height="10" fill="none" stroke="#8E44AD" stroke-width="2"/>
            <line x1="42" y1="40" x2="58" y2="40" stroke="#8E44AD" stroke-width="2"/>
        `;
    } else {
        // Pelo corto peinado, estilo contador prolijo
        hair = `
            <path d="M30 20 Q 50 10, 70 20 C 72 35, 72 50, 68 60 Q 50 70, 32 60 C 28 50, 28 35, 30 20 Z" fill="#3B2F2F"/>
        `;

        // Lentes tipo ejecutivos, más finos
        accessory = `
            <line x1="32" y1="40" x2="44" y2="40" stroke="#2C3E50" stroke-width="2"/>
            <line x1="56" y1="40" x2="68" y2="40" stroke="#2C3E50" stroke-width="2"/>
            <line x1="44" y1="40" x2="56" y2="40" stroke="#2C3E50" stroke-width="1"/>
        `;
    }

    break;
        
        case "microeconomía":
        case "economía":
            bodyBaseColor = '#3498DB'; // Un azul moderno y corporativo
            accessory = `
                <polygon points="50,60 47,75 53,75" fill="#2980B9"/>
                <path d="M40 30 L45 25 L50 30 L55 25 L60 30" stroke="#F1C40F" stroke-width="2" fill="none"/>
            `;
            // Anteojos de diseño moderno, finos y con un toque de color
            features = `
                <line x1="42" y1="35" x2="58" y2="35" stroke="#7F8C8D" stroke-width="1.5"/> <rect x="35" y="30" width="15" height="10" fill="rgba(0,0,0,0.1)" stroke="#7F8C8D" stroke-width="0.8" rx="2" ry="2"/>
                <rect x="50" y="30" width="15" height="10" fill="rgba(0,0,0,0.1)" stroke="#7F8C8D" stroke-width="0.8" rx="2" ry="2"/>
            `;

            if (professor.gender === "female") {
                // Pelo moderno con bob o corte simétrico
                hair = `<path d="M20 30 C 20 10, 80 10, 80 30 C 80 50, 70 70, 50 70 C 30 70, 20 50, 20 30 Z" fill="#34495E"/>`; // Gris azulado oscuro, moderno
            } else {
                // Pelo corto con textura o un flequillo moderno
                hair = `<path d="M30 20 Q 50 10 70 20 C 75 30, 75 50, 70 60 Q 50 70 30 60 C 25 50, 25 30, 30 20 Z" fill="#2C3E50"/>`; // Gris oscuro/negro moderno
            }
            break;

        case "gestión proyectos":
            bodyBaseColor = '#2ECC71'; // Un verde vibrante, asociado con crecimiento y éxito empresarial
            accessory = `
                <rect x="65" y="45" width="18" height="15" fill="#8B4513" rx="3" ry="3"/>
                <line x1="68" y1="45" x2="68" y2="60" stroke="#654321" stroke-width="1.5" />
                <path d="M40 25 L45 20 L55 20 L60 25 L55 30 L45 30 Z" fill="#F1C40F" stroke="#B7950B" stroke-width="1.5"/>
                <circle cx="42.5" cy="27.5" r="2" fill="#B7950B"/>
                <circle cx="57.5" cy="27.5" r="2" fill="#B7950B"/>
            `;
            // Lentes con un toque más moderno o robusto (si aplica)
            features = `
                <line x1="42" y1="35" x2="58" y2="35" stroke="#555" stroke-width="2"/>
                <rect x="35" y="30" width="15" height="10" fill="rgba(0,0,0,0.15)" stroke="#555" stroke-width="1"/>
                <rect x="50" y="30" width="15" height="10" fill="rgba(0,0,0,0.15)" stroke="#555" stroke-width="1"/>
            `;

            if (professor.gender === "female") {
                // Pelo profesional y dinámico para mujer
                hair = `<path d="M25 20 C 15 5, 15 70, 25 80 C 40 85, 60 85, 75 80 C 85 70, 85 5, 75 20 Z" fill="#6A0572"/>`; // Podría ser un recogido o corte moderno
            } else {
                // Pelo corto y bien peinado, clásico de empresario
                hair = `<path d="M30 20 Q 50 5 70 20 C 75 30, 75 50, 70 60 Q 50 75 30 60 C 25 50, 25 30, 30 20 Z" fill="#2C3E50"/>`; // Negro o muy oscuro
            }
            break;

    
        case "expresión oral y escrita":
        case "comprensión de textos":
            bodyBaseColor = '#9B59B6'; // Morado, un color que evoca creatividad y conocimiento.
            accessory = `
                <rect x="65" y="20" width="18" height="25" fill="#F0E68C" stroke="#8B4513" stroke-width="1.5" rx="2" ry="2"/>
                <text x="74" y="36" font-family="Arial" font-size="9" fill="#333" text-anchor="middle">📚</text>
                <line x1="42" y1="35" x2="58" y2="35" stroke="#333" stroke-width="1.5" stroke-linecap="round"/>
                <rect x="35" y="30" width="15" height="10" fill="rgba(0,0,0,0.2)" stroke="#333" stroke-width="1"/>
                <rect x="50" y="30" width="15" height="10" fill="rgba(0,0,0,0.2)" stroke="#333" stroke-width="1"/>
            `;
            // Los anteojos se superponen sobre los ojos definidos al final de la función
            features = ''; // Limpiamos cualquier otra feature si no es necesaria.

            if (professor.gender === "female") {
                // Nuevo peinado femenino más común y menos "raro"
                hair = `
                    <path d="M20 20 Q 50 0 80 20 C 85 40, 85 60, 80 80 H 20 C 15 60, 15 40, 20 20 Z" fill="#4B382D"/>
                    <path d="M25 25 C 20 35, 20 45, 25 55 C 30 65, 70 65, 75 55 C 80 45, 80 35, 75 25" fill="#4B382D" opacity="0.6"/>
                `; // Un pelo más voluminoso y menos "plano"
            } else {
                hair = `<path d="M30 20 Q 50 5 70 20 C 75 30, 75 50, 70 60 Q 50 75 30 60 C 25 50, 25 30, 30 20 Z" fill="#4B382D"/>`; // Pelo masculino genérico
            }
            break;

        case "historia occidental":
            bodyBaseColor = '#A0522D'; // Un color más terroso o histórico (marrón)
            accessory = `
                <circle cx="50" cy="25" r="10" fill="#F4D03F" stroke="#B7950B" stroke-width="2"/>
                <line x1="50" y1="20" x2="50" y2="30" stroke="#B7950B" stroke-width="2"/>
                <line x1="45" y1="25" x2="55" y2="25" stroke="#B7950B" stroke-width="2"/>
                <text x="50" y="45" font-family="Arial" font-size="12" fill="#333" text-anchor="middle">🏛️</text>
            `;
            features = '';
            if (professor.gender === "female") {
                // Pelo blanco para mujer
                hair = `<path d="M25 25 C 10 20, 10 55, 25 60 C 40 65, 60 65, 75 60 C 90 55, 90 20, 75 25" fill="#F5F5DC"/>`; // Blanco/Blanquecino
            } else {
                // Pelo blanco para hombre
                hair = `<path d="M30 20 Q 50 5 70 20 C 75 30, 75 50, 70 60 Q 50 75 30 60 C 25 50, 25 30, 30 20 Z" fill="#F5F5DC"/>`; // Blanco/Blanquecino
            }
            break;

        case "neurociencias": // NUEVO CASO PARA NEUROCIENCIAS (más intelectual, sin tapar la cara)
    bodyBaseColor = '#34495E'; // Un color gris/azul oscuro/morado, que puede evocar la ciencia o el pensamiento
    // Un símbolo más abstracto y "flotante" para la intelectualidad/pensamiento
    accessory = `
        <circle cx="50" cy="25" r="5" fill="#ADD8E6" /> <line x1="50" y1="25" x2="40" y2="15" stroke="#ADD8E6" stroke-width="1.5" stroke-linecap="round"/>
        <line x1="50" y1="25" x2="60" y2="15" stroke="#ADD8E6" stroke-width="1.5" stroke-linecap="round"/>
        <line x1="50" y1="25" x2="35" y2="35" stroke="#ADD8E6" stroke-width="1.5" stroke-linecap="round"/>
        <line x1="50" y1="25" x2="65" y2="35" stroke="#ADD8E6" stroke-width="1.5" stroke-linecap="round"/>
        <circle cx="40" cy="15" r="3" fill="#ADD8E6" />
        <circle cx="60" cy="15" r="3" fill="#ADD8E6" />
        <circle cx="35" cy="35" r="3" fill="#ADD8E6" />
        <circle cx="65" cy="35" r="3" fill="#ADD8E6" />
    `; // Simboliza conexiones neuronales o ideas interconectadas. Posicionado arriba de la cabeza.
    features = ''; // No necesitamos features extra aquí, el accessory ya es el foco.

    if (professor.gender === "female") {
        hair = `<path d="M25 25 C 10 20, 10 55, 25 60 C 40 65, 60 65, 75 60 C 90 55, 90 20, 75 25" fill="#A0522D"/>`; // Pelo femenino genérico
    } else {
        hair = `<path d="M30 20 Q 50 5 70 20 C 75 30, 75 50, 70 60 Q 50 75 30 60 C 25 50, 25 30, 30 20 Z" fill="#4B382D"/>`; // Pelo masculino genérico
    }
    break;
    default: // Para cualquier otra materia no definida explícitamente
        bodyBaseColor = professor.color || '#7F8C8D'; // Color por defecto si no hay uno definido
        if (professor.gender === "female") {
                hair = `<path d="M25 25 C 10 20, 10 55, 25 60 C 40 65, 60 65, 75 60 C 90 55, 90 20, 75 25" fill="#A0522D"/>`;
        } else {
            hair = `<path d="M30 20 Q 50 5 70 20 C 75 30, 75 50, 70 60 Q 50 75 30 60 C 25 50, 25 30, 30 20 Z" fill="#4B382D"/>`;
        }
        break;
}


const eyes = `<circle cx="40" cy="35" r="5" fill="#333"/><circle cx="60" cy="35" r="5" fill="#333"/>`;
const mouth = `<path d="M 40 50 Q 50 55 60 50" stroke="#333" stroke-width="3" fill="none"/>`;

return `
    <svg viewBox="0 0 100 100" style="width: 100%; height: 100%;">
        ${hair}
        <circle cx="50" cy="40" r="20" fill="${skinColor}"/>
        <rect x="30" y="60" width="40" height="30" rx="10" ry="10" fill="${bodyBaseColor}"/>
        ${eyes}
        ${mouth}
        ${accessory}
        ${features}
    </svg>
`;
}

// Pasaje de profesores (Estas funciones no necesitan cambios)
function nextAvatar() {
currentAvatarIndex = (currentAvatarIndex + 1) % professorAvatars.length;
}

function prevAvatar() {
currentAvatarIndex = (currentAvatarIndex - 1 + professorAvatars.length) % professorAvatars.length;
}
    // Cuando el componente se monta, asegura que el scroll-container tenga la propiedad de scroll snap
    onMount(() => {
        const scrollContainer = document.querySelector('.scroll-container');
        if (scrollContainer) {
            scrollContainer.style.scrollSnapType = 'y mandatory';
            scrollContainer.style.overflowY = 'scroll';
            scrollContainer.style.height = '100vh'; 
        }
    });
</script>

<svelte:head>
    <link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@400;700&display=swap" rel="stylesheet">
</svelte:head>

<div class="background-stars"></div>

<main class="scroll-container" on:scroll={handleScroll}>
    <section id="galaxy-intro" class="scroll-section">
        <h1 class="galaxy-title">
            La galaxia estelar <br> de mis profesores preferidos
        </h1>

        <div class="references-container" class:visible={showLegend}>
            <button class="toggle-legend-button" on:click={() => showLegend = !showLegend} aria-expanded={showLegend} aria-controls="legend-content">
                {showLegend ? 'Ocultar Leyenda' : 'Mostrar Leyenda'}
            </button>
            <div id="legend-content" class="legend-content">
                <div class="reference-section">
                    <span style="color: #ccc; font-size: 0.9em;">Mood</span>
                    <div class="mood-item">
                        <svg class="reference-star star-fun" viewBox="0 0 24 24" aria-hidden="true"><path d="M12 2L14.09 8.26H20.82L15.36 12.14L17.45 18.4L12 14.52L6.55 18.4L8.64 12.14L3.18 8.26H9.91L12 2Z"/></svg>
                        <span style="color: #00FFCC;">Divertido</span>
                    </div>
                    <div class="mood-item">
                        <svg class="reference-star star-interesting" viewBox="0 0 24 24" aria-hidden="true"><path d="M12 2L14.09 8.26H20.82L15.36 12.14L17.45 18.4L12 14.52L6.55 18.4L8.64 12.14L3.18 8.26H9.91L12 2Z"/></svg>
                        <span style="color: #FFD700;">Interesante</span>
                    </div>
                    <div class="mood-item">
                        <svg class="reference-star star-boring" viewBox="0 0 24 24" aria-hidden="true"><path d="M12 2L14.09 8.26H20.82L15.36 12.14L17.45 18.4L12 14.52L6.55 18.4L8.64 12.14L3.18 8.26H9.91L12 2Z"/></svg>
                        <span style="color: #FF69B4;">Aburrido</span>
                    </div>
                </div>

                <div class="reference-section">
                    <span style="color: #ccc; font-size: 0.9em;">Tamaño de Estrella (Like)</span>
                    <div class="like-item">
                        <svg class="reference-star" width="{starSize(10, undefined)}" height="{starSize(10, undefined)}" viewBox="0 0 24 24" style="fill: #eee;" aria-hidden="true"><path d="M12 2L14.09 8.26H20.82L15.36 12.14L17.45 18.4L12 14.52L6.55 18.4L8.64 12.14L3.18 8.26H9.91L12 2Z"/></svg>
                        <span style="color: green;">Gigante (10)</span>
                    </div>
                    <div class="like-item">
                        <svg class="reference-star" width="{starSize(9, undefined)}" height="{starSize(9, undefined)}" viewBox="0 0 24 24" style="fill: #eee;" aria-hidden="true"><path d="M12 2L14.09 8.26H20.82L15.36 12.14L17.45 18.4L12 14.52L6.55 18.4L8.64 12.14L3.18 8.26H9.91L12 2Z"/></svg>
                        <span style="color: lightgreen;">Grande (9)</span>
                    </div>
                    <div class="like-item">
                        <svg class="reference-star" width="{starSize(8, undefined)}" height="{starSize(8, undefined)}" viewBox="0 0 24 24" style="fill: #eee;" aria-hidden="true"><path d="M12 2L14.09 8.26H20.82L15.36 12.14L17.45 18.4L12 14.52L6.55 18.4L8.64 12.14L3.18 8.26H9.91L12 2Z"/></svg>
                        <span style="color: yellow;">Normal (8)</span>
                    </div>
                    <div class="like-item">
                        <svg class="reference-star" width="{starSize(7, undefined)}" height="{starSize(7, undefined)}" viewBox="0 0 24 24" style="fill: #eee;" aria-hidden="true"><path d="M12 2L14.09 8.26H20.82L15.36 12.14L17.45 18.4L12 14.52L6.55 18.4L8.64 12.14L3.18 8.26H9.91L12 2Z"/></svg>
                        <span style="color: orange;">Mediana (7)</span>
                    </div>
                    <div class="like-item">
                        <svg class="reference-star" width="{starSize(6, undefined)}" height="{starSize(6, undefined)}" viewBox="0 0 24 24" style="fill: #eee;" aria-hidden="true"><path d="M12 2L14.09 8.26H20.82L15.36 12.14L17.45 18.4L12 14.52L6.55 18.4L8.64 12.14L3.18 8.26H9.91L12 2Z"/></svg>
                        <span style="color: #f0c040;">Pequeña (6)</span>
                    </div>
                    <div class="like-item">
                        <svg class="reference-star" width="{starSize(2, undefined)}" height="{starSize(2, undefined)}" viewBox="0 0 24 24" style="fill: #eee;" aria-hidden="true"><path d="M12 2L14.09 8.26H20.82L15.36 12.14L17.45 18.4L12 14.52L6.55 18.4L8.64 12.14L3.18 8.26H9.91L12 2Z"/></svg>
                        <span style="color: red;">Mínima (&lt;6)</span>
                    </div>
                </div>
            </div>
        </div>

        <button class="top-professors-toggle" on:click={() => isTopProfessorsMode = !isTopProfessorsMode} aria-expanded={isTopProfessorsMode} aria-controls="top-professors-overlay">
            <svg width="24" height="24" viewBox="0 0 24 24" fill="#ffd700" aria-hidden="true"><path d="M12 17.27L18.18 21L16.54 13.97L22 9.24L14.81 8.62L12 2L9.19 8.62L2 9.24L7.46 13.97L5.82 21L12 17.27L18.18 21L16.54 13.97L22 9.24L14.81 8.62L12 2Z"/></svg>
            <span>Mi Top 4 de Profesores</span>
        </button>

        <div class="galaxy-container" class:hidden={isTopProfessorsMode}>
            {#each Object.entries(scoreYPositions) as [scoreGroup, yPos]}
                <div class="score-label" style="top: {yPos}vh;">
                    {scoreGroup === '<6' ? 'No Elegidos' : `Puntaje ${scoreGroup}`}
                </div>
            {/each}

            {#each displayedStars as star (star.id)}
            <button
            class="star-container {isFavorite(star) ? 'favorite-glow' : ''}"
            style="top: {star.y}%; left: {star.x}%" on:click={() => openProfessorDetails(star)}
            aria-label={`Ver detalles de ${star.name}, Puntaje ${star.like}`}
        >
                    <svg
                        class="star"
                        width="{starSize(star.like, undefined)}"
                        height="{starSize(star.like, undefined)}"
                        viewBox="0 0 24 24"
                        fill="{starColor(star)}"
                        xmlns="http://www.w3.org/2000/svg"
                        aria-hidden="true"
                    >
                        <path d="M12 2L14.09 8.26H20.82L15.36 12.14L17.45 18.4L12 14.52L6.55 18.4L8.64 12.14L3.18 8.26H9.91L12 2Z"/>
                    </svg>
                </button>
            {/each}
        </div>

        <div id="top-professors-overlay" class="ranked-stars-container" class:visible={isTopProfessorsMode}>
            {#each getRankedStars() as star (star.id)}
            <button
    class="star-container ranked-star {isFavorite(star) ? 'favorite-glow' : ''}"
    style="top: {star.y}%; left: {star.x}%" on:click={() => openProfessorDetails(star)}
    aria-label={`Ver detalles de ${star.name}, Ranking ${star.rank}`}
>
                    <svg class="star"
                        width="{starSize(star.like, star.rank)}"
                        height="{starSize(star.like, star.rank)}"
                        viewBox="0 0 24 24"
                        fill="{starColor(star)}"
                        xmlns="http://www.w3.org/2000/svg"
                        aria-hidden="true"
                    >
                        <path d="M12 2L14.09 8.26H20.82L15.36 12.14L17.45 18.4L12 14.52L6.55 18.4L8.64 12.14L3.18 8.26H9.91L12 2Z"/>
                    </svg>
                    <span class="rank-number">{star.rank}</span>
                </button>
            {/each}
        </div>

        <div class="controls">
            <button on:click={() => setFilter("all")} class:active={filter === "all" && filterSubject === "all"}>🌌 Todos</button>
            <button on:click={() => setFilter("liked")} class:active={filter === "liked"}>💚 Elegidos</button>
            <button on:click={() => setFilter("disliked")} class:active={filter === "disliked"}>💔 No Elegidos</button>
        </div>
    </section>

    <section id="avatar-section" class="scroll-section avatar-creator-section">
        <h2 class="section-title">Encuentra a tu profesor</h2>
        <p class="section-description">Aquí puedes ver los avatares de los profesores con una frase que lo define.</p>

        <div class="avatar-carousel-container">
            <button class="carousel-arrow left" on:click={prevAvatar}>&lt;</button>
            <div class="avatar-display-area">
                {#if professorAvatars[currentAvatarIndex]}
                    <!-- svelte-ignore a11y-click-events-have-key-events -->
                    <!-- svelte-ignore a11y-no-static-element-interactions -->
                    <div
                        class="professor-avatar-card"
                        on:click={() => openAvatarDetails(professorAvatars[currentAvatarIndex])}
                        aria-label={`Ver avatar de ${professorAvatars[currentAvatarIndex].name}`}
                    >
                        {@html generateProfessorAvatarSvg(professorAvatars[currentAvatarIndex])}
                        <h3>{professorAvatars[currentAvatarIndex].name}</h3>
                        <p>{professorAvatars[currentAvatarIndex].subject}</p>
                    </div>
                {/if}
            </div>
            <button class="carousel-arrow right" on:click={nextAvatar}>&gt;</button>
        </div>


        <section id="footer-section" class="scroll-section">
            <!-- svelte-ignore a11y-click-events-have-key-events (si no pongo esto me tira un warning) -->
            <footer class="main-footer">
                <!-- svelte-ignore a11y-no-static-element-interactions -->
                <div class="footer-button-content" on:click={() => showCreatorModal = true}>
                    <p class="footer-call-to-action">Conoce más de la creadora del diseño</p>
                    </div>
            </footer>
        </section>

        {#if showCreatorModal}
        <div class="modal-overlay" transition:fade|local>
            <div class="modal-content creator-modal-content">
                <button class="close-button" on:click={() => showCreatorModal = false}>X</button>
                <h2 class="creator-modal-title">Steffy</h2>
                <p class="creator-modal-text">Estudiante de Tecnología Digital con una pasión muy grande por el diseño web.</p>
                <p class="work">Trabajo para Visualización de Datos</p>
                <div class="creator-social-links">
                    <a href="https://www.instagram.com/stefanyred/" target="_blank" aria-label="Visita mi Instagram">
                        <svg class="instagram-icon-large" viewBox="0 0 24 24">
                            <path d="M12 2.163c3.204 0 3.584.012 4.85.07 3.252.148 4.771 1.691 4.919 4.919.058 1.265.07 1.646.07 4.85s-.012 3.584-.07 4.85c-.148 3.228-1.667 4.772-4.919 4.919-.058.029-.113.041-.24.052-.962.046-1.22.05-4.608.05-3.388 0-3.646-.004-4.608-.05-.127-.01-.182-.023-.24-.052-3.251-.147-4.771-1.692-4.919-4.919-.058-1.265-.07-1.646-.07-4.85s.012-3.584.07-4.85c.148-3.227 1.667-4.771 4.919-4.919.058-.029.113-.041.24-.052 1.066-.05 1.32-.054 4.608-.054zm0-.946c-3.606 0-4.042.016-5.16.066-4.35.211-6.19 2.062-6.402 6.402-.05 1.118-.066 1.554-.066 5.16s.016 4.042.066 5.16c.211 4.35 2.062 6.19 6.402 6.402 1.118.05 1.554.066 5.16.066s4.042-.016 5.16-.066c4.35-.211 6.19-2.062 6.402-6.402.05-1.118.066-1.554.066-5.16s-.016-4.042-.066-5.16c-.211-4.35-2.062-6.19-6.402-6.402-1.118-.05-1.554-.066-5.16-.066zm0 5.86c-2.341 0-4.241 1.9-4.241 4.241s1.9 4.241 4.241 4.241 4.241-1.9 4.241-4.241-1.9-4.241-4.241-4.241zm0 7.382c-1.734 0-3.141-1.407-3.141-3.141s1.407-3.141 3.141-3.141 3.141 1.407 3.141 3.141-1.407 3.141-3.141 3.141zm6.406-9.146c-.638 0-1.157.519-1.157 1.157s.519 1.157 1.157 1.157 1.157-.519 1.157-1.157-.519-1.157-1.157-1.157z" fill="currentColor"/>
                        </svg>
                    </a>
                </div>
            </div>
        </div>
    {/if}

   

</main>

{#if selectedProfessor}
    <div class="modal-overlay" transition:fade|local>
        <div class="modal-content">
            <button class="close-button" on:click={closeProfessorDetails}>X</button>
            <h2>{selectedProfessor.name}</h2>
            <p><strong>Materia:</strong> {selectedProfessor.subject}</p>
            <p><strong>Año en que la curse:</strong> {selectedProfessor.year}</p>
            <p><strong>Mi Nota (Like):</strong> {selectedProfessor.like}</p>
            </div>
    </div>
{/if}

{#if selectedAvatarProfessor}
    <div class="modal-overlay" transition:fade|local>
        <div class="modal-content avatar-modal-content">
            <button class="close-button" on:click={closeAvatarDetails}>X</button>
            <div class="avatar-detail-display">
                {@html generateProfessorAvatarSvg(selectedAvatarProfessor)}
            </div>
            <div class="avatar-info">
                <h3>{selectedAvatarProfessor.name}</h3>
                <p><strong>Materia:</strong> {selectedAvatarProfessor.subject}</p>
                {#if selectedAvatarProfessor.description}
                    <p>{selectedAvatarProfessor.description}</p>
                {/if}
                <p class="professor-phrase">
                    {#if selectedAvatarProfessor.phrase}
                        "{selectedAvatarProfessor.phrase}"
                    {:else}
                        No hay una frase definida para este profesor.
                    {/if}
                </p>
            </div>
        </div>
    </div>
{/if}