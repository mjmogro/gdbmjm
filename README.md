<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Geoportal de Observaciones de Tortugas Marinas</title>
    
    <!-- Leaflet CSS -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/leaflet/1.9.4/leaflet.css" />
    
    <!-- Font Awesome -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css">
    
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
            height: 100vh;
            overflow: hidden;
            background: #f0f2f5;
        }

        #map {
            height: 100vh;
            width: 100%;
            z-index: 1;
        }

        /* Panel de Control */
        .control-panel {
            position: absolute;
            top: 20px;
            left: 20px;
            width: 340px;
            max-height: 90vh;
            background: rgba(255, 255, 255, 0.95);
            backdrop-filter: blur(10px);
            border-radius: 16px;
            box-shadow: 0 8px 32px rgba(0, 0, 0, 0.12);
            z-index: 1000;
            overflow-y: auto;
            overflow-x: hidden;
            padding: 12px;
        }

        .control-panel::-webkit-scrollbar {
            width: 8px;
        }

        .control-panel::-webkit-scrollbar-track {
            background: rgba(0, 0, 0, 0.05);
            border-radius: 10px;
        }

        .control-panel::-webkit-scrollbar-thumb {
            background: rgba(78, 205, 196, 0.5);
            border-radius: 10px;
        }

        /* Header Principal */
        .panel-main-header {
            text-align: center;
            margin-bottom: 15px;
            padding-bottom: 15px;
            border-bottom: 2px solid #e0e0e0;
        }

        .panel-main-header h1 {
            font-size: 20px;
            font-weight: 700;
            color: #2c3e50;
            margin-bottom: 3px;
        }

        .panel-main-header p {
            font-size: 12px;
            color: #7f8c8d;
        }

        /* Secciones con estilo de botón */
        .section {
            margin-bottom: 8px;
        }

        .section-header {
            background: linear-gradient(135deg, #52b788 0%, #40916c 100%);
            color: white;
            padding: 12px 16px;
            border-radius: 10px;
            cursor: pointer;
            display: flex;
            align-items: center;
            justify-content: space-between;
            transition: all 0.3s ease;
            user-select: none;
            box-shadow: 0 2px 6px rgba(82, 183, 136, 0.25);
        }

        .section-header:hover {
            transform: translateY(-1px);
            box-shadow: 0 3px 10px rgba(82, 183, 136, 0.35);
        }

        .section-header:active {
            transform: translateY(0);
        }

        .section-header.active {
            border-radius: 10px 10px 0 0;
            margin-bottom: 0;
        }

        .section-header-content {
            display: flex;
            align-items: center;
            gap: 10px;
            font-size: 14px;
            font-weight: 600;
        }

        .section-header-content i {
            font-size: 16px;
            width: 20px;
            text-align: center;
        }

        .section-header .arrow {
            transition: transform 0.3s ease;
            font-size: 14px;
        }

        .section-header.active .arrow {
            transform: rotate(180deg);
        }

        .section-content {
            max-height: 0;
            overflow: hidden;
            transition: max-height 0.3s ease;
            background: white;
            border: 1px solid #e0e0e0;
            border-top: none;
            border-radius: 0 0 10px 10px;
        }

        .section-content.active {
            max-height: 800px;
            padding: 15px;
            margin-bottom: 8px;
            box-shadow: 0 2px 6px rgba(0, 0, 0, 0.06);
            overflow-y: auto;
        }

        /* Estadísticas */
        .stats-grid {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 15px;
        }

        .stat-card {
            background: #f8f9fa;
            padding: 20px;
            border-radius: 10px;
            text-align: center;
            border: 1px solid #e9ecef;
            transition: all 0.3s ease;
        }

        .stat-card:hover {
            transform: translateY(-2px);
            box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
        }

        .stat-card .value {
            font-size: 36px;
            font-weight: 700;
            color: #52b788;
            margin-bottom: 5px;
        }

        .stat-card .label {
            font-size: 12px;
            color: #6c757d;
            text-transform: uppercase;
            letter-spacing: 0.5px;
        }

        /* Filtros de Especies */
        .species-filters {
            display: flex;
            flex-direction: column;
            gap: 10px;
        }

        .species-btn {
            padding: 12px 16px;
            border: 2px solid;
            border-radius: 10px;
            font-size: 14px;
            font-weight: 600;
            cursor: pointer;
            transition: all 0.3s ease;
            display: flex;
            align-items: center;
            gap: 10px;
            background: white;
            position: relative;
            overflow: hidden;
        }

        .species-btn:hover {
            transform: translateX(5px);
        }

        .species-btn.inactive {
            opacity: 0.5;
            background: #f8f9fa;
        }

        .species-btn .indicator {
            width: 12px;
            height: 12px;
            border-radius: 50%;
            flex-shrink: 0;
        }

        .species-btn .count {
            margin-left: auto;
            background: rgba(0, 0, 0, 0.08);
            padding: 2px 10px;
            border-radius: 12px;
            font-size: 12px;
            color: #495057;
        }

        /* Colores de Especies */
        .species-verde { 
            border-color: #27ae60;
            color: #27ae60;
        }
        .species-verde .indicator { background: #27ae60; }

        .species-carey { 
            border-color: #e67e22;
            color: #e67e22;
        }
        .species-carey .indicator { background: #e67e22; }

        .species-boba { 
            border-color: #3498db;
            color: #3498db;
        }
        .species-boba .indicator { background: #3498db; }

        .species-laud { 
            border-color: #9b59b6;
            color: #9b59b6;
        }
        .species-laud .indicator { background: #9b59b6; }

        .species-golfina { 
            border-color: #f39c12;
            color: #f39c12;
        }
        .species-golfina .indicator { background: #f39c12; }

        /* Switches iOS */
        .layer-controls {
            display: flex;
            flex-direction: column;
            gap: 15px;
        }

        .layer-item {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 12px 16px;
            background: #f8f9fa;
            border-radius: 10px;
            transition: all 0.3s ease;
        }

        .layer-item:hover {
            background: #e9ecef;
        }

        .layer-item label {
            font-size: 14px;
            color: #495057;
            font-weight: 500;
            display: flex;
            align-items: center;
            gap: 10px;
        }

        .layer-item label i {
            color: #52b788;
            width: 20px;
        }

        .ios-switch {
            position: relative;
            width: 51px;
            height: 31px;
            cursor: pointer;
        }

        .ios-switch input {
            opacity: 0;
            width: 0;
            height: 0;
        }

        .ios-switch-bg {
            position: absolute;
            top: 0;
            left: 0;
            right: 0;
            bottom: 0;
            background: #e0e0e0;
            border-radius: 31px;
            transition: background 0.3s;
        }

        .ios-switch .slider {
            position: absolute;
            top: 2px;
            left: 2px;
            width: 27px;
            height: 27px;
            background: white;
            border-radius: 50%;
            transition: transform 0.3s;
            box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
        }

        .ios-switch input:checked + .ios-switch-bg {
            background: #52b788;
        }

        .ios-switch input:checked ~ .slider {
            transform: translateX(20px);
        }

        /* Lista de Observaciones */
        .observations-list {
            max-height: 300px;
            overflow-y: auto;
        }

        .observation-item {
            background: #f8f9fa;
            padding: 12px;
            border-radius: 8px;
            margin-bottom: 8px;
            cursor: pointer;
            transition: all 0.3s ease;
            border: 2px solid transparent;
        }

        .observation-item:hover {
            border-color: #52b788;
            transform: translateX(5px);
        }

        .observation-item.active {
            border-color: #52b788;
            background: #e8f5e9;
        }

        .observation-item h4 {
            margin: 0 0 8px;
            color: #2c3e50;
            font-size: 16px;
            display: flex;
            align-items: center;
            gap: 8px;
        }

        .observation-item .species-dot {
            width: 12px;
            height: 12px;
            border-radius: 50%;
            flex-shrink: 0;
        }

        .observation-item p {
            margin: 4px 0;
            font-size: 13px;
            color: #6c757d;
        }

        .observation-item .date {
            font-size: 12px;
            color: #adb5bd;
        }

        /* Formulario */
        .form-group {
            margin-bottom: 16px;
        }

        .form-group label {
            display: block;
            font-size: 13px;
            color: #495057;
            margin-bottom: 6px;
            font-weight: 600;
        }

        .form-control {
            width: 100%;
            padding: 10px 14px;
            border: 2px solid #e0e0e0;
            border-radius: 8px;
            font-size: 14px;
            transition: all 0.3s ease;
            background: white;
        }

        .form-control:focus {
            outline: none;
            border-color: #52b788;
            box-shadow: 0 0 0 3px rgba(82, 183, 136, 0.1);
        }

        select.form-control {
            cursor: pointer;
        }

        textarea.form-control {
            resize: vertical;
            min-height: 80px;
        }

        .coords-group {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 10px;
        }

        /* Botones */
        .btn {
            padding: 12px 20px;
            border: none;
            border-radius: 8px;
            font-size: 14px;
            font-weight: 600;
            cursor: pointer;
            transition: all 0.3s ease;
            display: inline-flex;
            align-items: center;
            justify-content: center;
            gap: 8px;
            width: 100%;
        }

        .btn-primary {
            background: #52b788;
            color: white;
        }

        .btn-primary:hover {
            background: #40916c;
            transform: translateY(-1px);
            box-shadow: 0 4px 12px rgba(82, 183, 136, 0.3);
        }

        .btn-secondary {
            background: #e9ecef;
            color: #495057;
        }

        .btn-secondary:hover {
            background: #dee2e6;
        }

        .btn-gps {
            background: #3498db;
            color: white;
        }

        .btn-gps:hover {
            background: #2980b9;
        }

        /* Mensajes */
        .message {
            padding: 12px 16px;
            border-radius: 8px;
            margin-top: 12px;
            font-size: 13px;
            display: none;
            animation: slideIn 0.3s ease;
        }

        @keyframes slideIn {
            from {
                opacity: 0;
                transform: translateY(-10px);
            }
            to {
                opacity: 1;
                transform: translateY(0);
            }
        }

        .message.success {
            background: #d1fae5;
            color: #065f46;
            border: 1px solid #6ee7b7;
        }

        .message.error {
            background: #fee2e2;
            color: #991b1b;
            border: 1px solid #fca5a5;
        }

        /* Acciones */
        .actions-grid {
            display: grid;
            gap: 10px;
        }

        /* Estado de conexión */
        .connection-status {
            position: absolute;
            top: 20px;
            right: 20px;
            padding: 8px 16px;
            border-radius: 20px;
            font-size: 12px;
            font-weight: 600;
            box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
            z-index: 1000;
            display: flex;
            align-items: center;
            gap: 8px;
        }

        .connection-status.online {
            background: #d1fae5;
            color: #065f46;
        }

        .connection-status.offline {
            background: #fee2e2;
            color: #991b1b;
        }

        .connection-status .indicator {
            width: 8px;
            height: 8px;
            border-radius: 50%;
            animation: pulse 2s infinite;
        }

        .connection-status.online .indicator {
            background: #10b981;
        }

        .connection-status.offline .indicator {
            background: #ef4444;
        }

        @keyframes pulse {
            0% { opacity: 1; }
            50% { opacity: 0.5; }
            100% { opacity: 1; }
        }

        /* Responsive */
        @media (max-width: 768px) {
            .control-panel {
                width: calc(100% - 20px);
                left: 10px;
                right: 10px;
                max-height: 85vh;
            }

            .panel-main-header h1 {
                font-size: 18px;
            }

            .coords-group {
                grid-template-columns: 1fr;
            }

            .stats-grid {
                grid-template-columns: 1fr;
            }

            .connection-status {
                top: auto;
                bottom: 20px;
                right: 20px;
            }
        }

        /* Loading spinner */
        .spinner {
            display: inline-block;
            width: 16px;
            height: 16px;
            border: 2px solid rgba(255, 255, 255, 0.3);
            border-radius: 50%;
            border-top-color: white;
            animation: spin 0.8s linear infinite;
        }

        @keyframes spin {
            to { transform: rotate(360deg); }
        }

        /* Popup personalizado */
        .leaflet-popup-content-wrapper {
            border-radius: 12px;
            box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
        }

        .leaflet-popup-content {
            margin: 13px 19px;
            line-height: 1.5;
        }

        /* Marcador resaltado */
        .highlighted-marker {
            animation: pulse-marker 1s ease-in-out 3;
        }

        @keyframes pulse-marker {
            0% {
                stroke-width: 3;
                stroke-opacity: 1;
            }
            50% {
                stroke-width: 15;
                stroke-opacity: 0.5;
            }
            100% {
                stroke-width: 3;
                stroke-opacity: 1;
            }
        }
    </style>
</head>
<body>
    <div id="map"></div>
    
    <!-- Indicador de estado de conexión -->
    <div class="connection-status" id="connectionStatus">
        <span class="indicator"></span>
        <span id="connectionText">Verificando conexión...</span>
    </div>
    
    <div class="control-panel">
        <div class="panel-main-header">
            <h1>🐢 Geoportal de Tortugas Marinas</h1>
            <p>Sistema de monitoreo y observación</p>
        </div>

        <!-- Sección de Estadísticas -->
        <div class="section">
            <div class="section-header active" onclick="toggleSection(this)">
                <div class="section-header-content">
                    <i class="fas fa-chart-bar"></i>
                    <span>Estadísticas</span>
                </div>
                <i class="fas fa-chevron-down arrow"></i>
            </div>
            <div class="section-content active">
                <div class="stats-grid">
                    <div class="stat-card">
                        <div class="value" id="totalObservaciones">0</div>
                        <div class="label">Total Observaciones</div>
                    </div>
                    <div class="stat-card">
                        <div class="value" id="especiesUnicas">0</div>
                        <div class="label">Especies Únicas</div>
                    </div>
                </div>
            </div>
        </div>

        <!-- Sección de Filtros por Especie -->
        <div class="section">
            <div class="section-header" onclick="toggleSection(this)">
                <div class="section-header-content">
                    <i class="fas fa-filter"></i>
                    <span>Filtros por Especie</span>
                </div>
                <i class="fas fa-chevron-down arrow"></i>
            </div>
            <div class="section-content">
                <div class="species-filters">
                    <button class="species-btn species-verde" data-species="Verde" onclick="toggleSpeciesFilter(this)">
                        <span class="indicator"></span>
                        <span>Tortuga Verde</span>
                        <span class="count">0</span>
                    </button>
                    <button class="species-btn species-carey" data-species="Carey" onclick="toggleSpeciesFilter(this)">
                        <span class="indicator"></span>
                        <span>Tortuga Carey</span>
                        <span class="count">0</span>
                    </button>
                    <button class="species-btn species-boba" data-species="Boba" onclick="toggleSpeciesFilter(this)">
                        <span class="indicator"></span>
                        <span>Tortuga Boba</span>
                        <span class="count">0</span>
                    </button>
                    <button class="species-btn species-laud" data-species="Laúd" onclick="toggleSpeciesFilter(this)">
                        <span class="indicator"></span>
                        <span>Tortuga Laúd</span>
                        <span class="count">0</span>
                    </button>
                    <button class="species-btn species-golfina" data-species="Golfina" onclick="toggleSpeciesFilter(this)">
                        <span class="indicator"></span>
                        <span>Tortuga Golfina</span>
                        <span class="count">0</span>
                    </button>
                </div>
            </div>
        </div>

        <!-- Sección de Observaciones Registradas -->
        <div class="section">
            <div class="section-header" onclick="toggleSection(this)">
                <div class="section-header-content">
                    <i class="fas fa-list"></i>
                    <span>Observaciones Registradas</span>
                </div>
                <i class="fas fa-chevron-down arrow"></i>
            </div>
            <div class="section-content">
                <div class="observations-list" id="observationsList">
                    <!-- Se llenará dinámicamente -->
                </div>
            </div>
        </div>

        <!-- Sección de Capas Adicionales -->
        <div class="section">
            <div class="section-header" onclick="toggleSection(this)">
                <div class="section-header-content">
                    <i class="fas fa-layer-group"></i>
                    <span>Capas Adicionales</span>
                </div>
                <i class="fas fa-chevron-down arrow"></i>
            </div>
            <div class="section-content">
                <div class="layer-controls">
                    <div class="layer-item">
                        <label><i class="fas fa-city"></i> Poblados</label>
                        <div class="ios-switch">
                            <input type="checkbox" id="togglePoblados" onchange="toggleLayer('poblados')">
                            <div class="ios-switch-bg"></div>
                            <div class="slider"></div>
                        </div>
                    </div>
                    <div class="layer-item">
                        <label><i class="fas fa-road"></i> Vías</label>
                        <div class="ios-switch">
                            <input type="checkbox" id="toggleVias" onchange="toggleLayer('vias')">
                            <div class="ios-switch-bg"></div>
                            <div class="slider"></div>
                        </div>
                    </div>
                    <div class="layer-item">
                        <label><i class="fas fa-building"></i> Zonas Urbanas</label>
                        <div class="ios-switch">
                            <input type="checkbox" id="toggleZonas" onchange="toggleLayer('zonas')">
                            <div class="ios-switch-bg"></div>
                            <div class="slider"></div>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <!-- Sección de Nueva Observación -->
        <div class="section">
            <div class="section-header" onclick="toggleSection(this)">
                <div class="section-header-content">
                    <i class="fas fa-plus-circle"></i>
                    <span>Nueva Observación</span>
                </div>
                <i class="fas fa-chevron-down arrow"></i>
            </div>
            <div class="section-content">
                <form id="observationForm">
                    <div class="form-group">
                        <label>Nombre del Observador</label>
                        <input type="text" class="form-control" id="nombreObservador" required>
                    </div>

                    <div class="form-group">
                        <label>Tag ID de la Tortuga</label>
                        <input type="text" class="form-control" id="tagId" placeholder="Ej: TM-001">
                    </div>

                    <div class="form-group">
                        <label>Tipo de Tortuga</label>
                        <select class="form-control" id="tipoTortuga" required>
                            <option value="">Seleccionar especie...</option>
                            <option value="Verde">Tortuga Verde</option>
                            <option value="Carey">Tortuga Carey</option>
                            <option value="Boba">Tortuga Boba</option>
                            <option value="Laúd">Tortuga Laúd</option>
                            <option value="Golfina">Tortuga Golfina</option>
                        </select>
                    </div>

                    <div class="form-group">
                        <label>Actividad</label>
                        <select class="form-control" id="actividad" required>
                            <option value="">Seleccionar actividad...</option>
                            <option value="Anidación">Anidación</option>
                            <option value="Alimentación">Alimentación</option>
                            <option value="Descanso">Descanso</option>
                            <option value="Migración">Migración</option>
                            <option value="Otro">Otro</option>
                        </select>
                    </div>

                    <div class="form-group">
                        <label>Observaciones</label>
                        <textarea class="form-control" id="observaciones" placeholder="Detalles adicionales..."></textarea>
                    </div>

                    <div class="form-group">
                        <label>Coordenadas</label>
                        <div class="coords-group">
                            <input type="number" class="form-control" id="latitud" placeholder="Latitud" step="0.000001" min="-90" max="90" required>
                            <input type="number" class="form-control" id="longitud" placeholder="Longitud" step="0.000001" min="-180" max="180" required>
                        </div>
                    </div>

                    <button type="button" class="btn btn-gps" onclick="obtenerUbicacion()">
                        <i class="fas fa-location-dot"></i> Obtener mi ubicación
                    </button>

                    <div id="formMessage" class="message"></div>

                    <button type="submit" class="btn btn-primary" style="margin-top: 16px;">
                        <i class="fas fa-save"></i> Guardar Observación
                    </button>
                </form>
            </div>
        </div>

        <!-- Sección de Acciones -->
        <div class="section">
            <div class="section-header" onclick="toggleSection(this)">
                <div class="section-header-content">
                    <i class="fas fa-cog"></i>
                    <span>Acciones</span>
                </div>
                <i class="fas fa-chevron-down arrow"></i>
            </div>
            <div class="section-content">
                <div class="actions-grid">
                    <button class="btn btn-secondary" onclick="actualizarDatos()">
                        <i class="fas fa-sync"></i> Actualizar Datos
                    </button>
                    <button class="btn btn-secondary" onclick="centrarMapa()">
                        <i class="fas fa-crosshairs"></i> Centrar Mapa
                    </button>
                </div>
            </div>
        </div>
    </div>

    <!-- Leaflet JS -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/leaflet/1.9.4/leaflet.js"></script>
    
    <!-- Supabase Client -->
    <script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>
    
    <script>
        // Configuración de Supabase con tus credenciales
        const SUPABASE_URL = 'https://dhvlvfhhagicnvfekxgc.supabase.co'
        const SUPABASE_ANON_KEY = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImRodmx2ZmhoYWdpY252ZmVreGdjIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NTAyNzYwNjIsImV4cCI6MjA2NTg1MjA2Mn0.U29qJQeU41AZkJh7Xm6Fj7J4wBo-UK_nqkhhtIpWCNQ'
        
        // Variables globales
        let supabaseClient = null;
        let isOfflineMode = false;
        let map;
        let observacionesLayer;
        let pobladosLayer;
        let viasLayer;
        let zonasLayer;
        let observaciones = [];
        let marcadores = {};
        let marcadorActivo = null;
        let filtrosActivos = {
            Verde: true,
            Carey: true,
            Boba: true,
            Laúd: true,
            Golfina: true
        };

        // Inicializar Supabase
        async function initSupabase() {
            try {
                const { createClient } = supabase;
                supabaseClient = createClient(SUPABASE_URL, SUPABASE_ANON_KEY);
                
                // Verificar conexión con una consulta simple
                const { data, error } = await supabaseClient
                    .from('observaciones_tortugas')
                    .select('id')
                    .limit(1);
                
                if (error) throw error;
                
                isOfflineMode = false;
                updateConnectionStatus(true);
                console.log('✅ Conexión con Supabase establecida');
            } catch (error) {
                console.log('⚠️ Error conectando con Supabase:', error.message);
                console.log('📴 Activando modo offline');
                isOfflineMode = true;
                updateConnectionStatus(false);
            }
        }

        // Actualizar estado de conexión
        function updateConnectionStatus(online) {
            const statusEl = document.getElementById('connectionStatus');
            const textEl = document.getElementById('connectionText');
            
            if (online) {
                statusEl.className = 'connection-status online';
                textEl.textContent = 'Conectado a Supabase';
            } else {
                statusEl.className = 'connection-status offline';
                textEl.textContent = 'Modo Offline';
            }
        }

        // Colores por especie
        const coloresEspecies = {
            Verde: '#27ae60',
            Carey: '#e67e22',
            Boba: '#3498db',
            Laúd: '#9b59b6',
            Golfina: '#f39c12'
        };

        // Datos de ejemplo para modo offline
        const observacionesEjemplo = [
            {
                id: 1,
                nombre_observador: 'María González',
                tag_id_tortuga: 'TM-001',
                tipo_tortuga: 'Verde',
                actividad: 'Anidación',
                observaciones: 'Tortuga saludable, puso aproximadamente 120 huevos',
                latitud: -0.9553,
                longitud: -80.7339,
                created_at: new Date().toISOString()
            },
            {
                id: 2,
                nombre_observador: 'Carlos Mendoza',
                tag_id_tortuga: 'TM-002',
                tipo_tortuga: 'Carey',
                actividad: 'Alimentación',
                observaciones: 'Observada alimentándose de esponjas marinas',
                latitud: -1.2621,
                longitud: -80.8547,
                created_at: new Date().toISOString()
            },
            {
                id: 3,
                nombre_observador: 'Ana Rodríguez',
                tag_id_tortuga: 'TM-003',
                tipo_tortuga: 'Laúd',
                actividad: 'Migración',
                observaciones: 'Tortuga en ruta migratoria hacia el norte',
                latitud: -0.6267,
                longitud: -80.4123,
                created_at: new Date().toISOString()
            },
            {
                id: 4,
                nombre_observador: 'Diego Vargas',
                tag_id_tortuga: 'TM-004',
                tipo_tortuga: 'Golfina',
                actividad: 'Descanso',
                observaciones: 'Descansando en superficie, comportamiento normal',
                latitud: -1.5434,
                longitud: -80.9876,
                created_at: new Date().toISOString()
            },
            {
                id: 5,
                nombre_observador: 'Laura Jiménez',
                tag_id_tortuga: 'TM-005',
                tipo_tortuga: 'Boba',
                actividad: 'Anidación',
                observaciones: 'Primera anidación de la temporada en esta playa',
                latitud: -0.8976,
                longitud: -80.6543,
                created_at: new Date().toISOString()
            }
        ];

        // Inicializar el mapa
        function initMap() {
            map = L.map('map').setView([-1.0, -80.7], 8);

            // Capas base
            const osm = L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
                attribution: '© OpenStreetMap contributors'
            });

            const satellite = L.tileLayer('https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}', {
                attribution: '© Esri'
            });

            const topographic = L.tileLayer('https://{s}.tile.opentopomap.org/{z}/{x}/{y}.png', {
                attribution: '© OpenTopoMap'
            });

            const light = L.tileLayer('https://{s}.basemaps.cartocdn.com/light_all/{z}/{x}/{y}{r}.png', {
                attribution: '© CartoDB'
            });

            // Agregar capa por defecto
            osm.addTo(map);

            // Control de capas base
            const baseMaps = {
                "Estándar": osm,
                "Satélite": satellite,
                "Topográfico": topographic,
                "Claro": light
            };

            L.control.layers(baseMaps).addTo(map);

            // Inicializar capas
            observacionesLayer = L.layerGroup().addTo(map);
            pobladosLayer = L.layerGroup();
            viasLayer = L.layerGroup();
            zonasLayer = L.layerGroup();

            // Cargar datos
            cargarDatos();
        }

        // Cargar todos los datos
        async function cargarDatos() {
            await cargarObservaciones();
            await cargarCapasAdicionales();
            actualizarEstadisticas();
            actualizarListaObservaciones();
        }

        // Cargar observaciones
        async function cargarObservaciones() {
            try {
                if (isOfflineMode) {
                    observaciones = observacionesEjemplo;
                    console.log('📱 Usando datos de ejemplo (modo offline)');
                } else {
                    const { data, error } = await supabaseClient
                        .from('observaciones_tortugas')
                        .select('*')
                        .order('created_at', { ascending: false });

                    if (error) throw error;
                    
                    observaciones = data || [];
                    console.log(`✅ ${observaciones.length} observaciones cargadas desde Supabase`);
                    
                    // Si no hay datos, usar ejemplos
                    if (observaciones.length === 0) {
                        console.log('ℹ️ No hay datos en la base de datos, usando ejemplos');
                        observaciones = observacionesEjemplo;
                    }
                }

                mostrarObservaciones();
            } catch (error) {
                console.error('❌ Error cargando observaciones:', error);
                observaciones = observacionesEjemplo;
                isOfflineMode = true;
                updateConnectionStatus(false);
                mostrarObservaciones();
            }
        }

        // Mostrar observaciones en el mapa
        function mostrarObservaciones() {
            observacionesLayer.clearLayers();
            marcadores = {};

            observaciones.forEach(obs => {
                if (filtrosActivos[obs.tipo_tortuga]) {
                    const marker = L.circleMarker([obs.latitud, obs.longitud], {
                        radius: 10,
                        fillColor: coloresEspecies[obs.tipo_tortuga],
                        color: '#fff',
                        weight: 3,
                        opacity: 1,
                        fillOpacity: 0.8
                    });

                    const fecha = new Date(obs.created_at).toLocaleDateString('es-EC');
                    const hora = new Date(obs.created_at).toLocaleTimeString('es-EC', { hour: '2-digit', minute: '2-digit' });

                    marker.bindPopup(`
                        <div style="min-width: 250px;">
                            <h4 style="margin: 0 0 10px; color: ${coloresEspecies[obs.tipo_tortuga]};">
                                <i class="fas fa-water"></i> ${obs.tipo_tortuga}
                            </h4>
                            <p style="margin: 5px 0;"><strong>Observador:</strong> ${obs.nombre_observador}</p>
                            <p style="margin: 5px 0;"><strong>Tag ID:</strong> ${obs.tag_id_tortuga || 'No registrado'}</p>
                            <p style="margin: 5px 0;"><strong>Actividad:</strong> ${obs.actividad}</p>
                            <p style="margin: 5px 0;"><strong>Observaciones:</strong> ${obs.observaciones || 'Sin observaciones'}</p>
                            <p style="margin: 5px 0;"><strong>Fecha:</strong> ${fecha} ${hora}</p>
                            <p style="margin: 5px 0;"><strong>Coordenadas:</strong> ${obs.latitud.toFixed(6)}, ${obs.longitud.toFixed(6)}</p>
                        </div>
                    `);

                    marker.addTo(observacionesLayer);
                    marcadores[obs.id] = marker;
                }
            });
        }

        // Actualizar lista de observaciones
        function actualizarListaObservaciones() {
            const lista = document.getElementById('observationsList');
            lista.innerHTML = '';

            observaciones.forEach(obs => {
                const fecha = new Date(obs.created_at).toLocaleDateString('es-EC');
                const hora = new Date(obs.created_at).toLocaleTimeString('es-EC', { hour: '2-digit', minute: '2-digit' });

                const item = document.createElement('div');
                item.className = 'observation-item';
                item.id = `obs-item-${obs.id}`;
                item.onclick = () => focusObservation(obs.id);

                item.innerHTML = `
                    <h4>
                        <span class="species-dot" style="background: ${coloresEspecies[obs.tipo_tortuga]}"></span>
                        ${obs.tipo_tortuga} - ${obs.tag_id_tortuga || 'Sin tag'}
                    </h4>
                    <p><strong>Observador:</strong> ${obs.nombre_observador}</p>
                    <p><strong>Actividad:</strong> ${obs.actividad}</p>
                    <p class="date">${fecha} ${hora}</p>
                `;

                lista.appendChild(item);
            });

            if (observaciones.length === 0) {
                lista.innerHTML = '<p style="text-align: center; color: #6c757d;">No hay observaciones registradas</p>';
            }
        }

        // Función para enfocar una observación
        function focusObservation(obsId) {
            const obs = observaciones.find(o => o.id === obsId);
            if (obs && marcadores[obsId]) {
                // Remover clase activa de todos los items
                document.querySelectorAll('.observation-item').forEach(item => {
                    item.classList.remove('active');
                });
                
                // Agregar clase activa al item seleccionado
                document.getElementById(`obs-item-${obsId}`).classList.add('active');
                
                // Centrar mapa en la observación
                map.setView([obs.latitud, obs.longitud], 14);
                
                // Abrir popup
                marcadores[obsId].openPopup();
                
                // Agregar animación al marcador
                const marker = marcadores[obsId];
                marker._path.classList.add('highlighted-marker');
                setTimeout(() => {
                    marker._path.classList.remove('highlighted-marker');
                }, 3000);
            }
        }

        // Cargar capas adicionales
        async function cargarCapasAdicionales() {
            if (isOfflineMode) {
                console.log('ℹ️ Capas adicionales no disponibles en modo offline');
                // Deshabilitar los switches
                document.querySelectorAll('.ios-switch input').forEach(input => {
                    input.disabled = true;
                    input.parentElement.style.opacity = '0.5';
                });
                return;
            }

            try {
                // Cargar poblados
                const { data: poblados, error: errorPoblados } = await supabaseClient
                    .from('poblados')
                    .select('*');

                if (!errorPoblados && poblados && poblados.length > 0) {
                    console.log(`✅ ${poblados.length} poblados cargados`);
                    poblados.forEach(poblado => {
                        const lat = poblado.latitud || poblado.lat;
                        const lng = poblado.longitud || poblado.lng || poblado.lon;
                        
                        if (lat && lng) {
                            const marker = L.circleMarker([lat, lng], {
                                radius: 6,
                                fillColor: '#e74c3c',
                                color: '#fff',
                                weight: 2,
                                opacity: 1,
                                fillOpacity: 0.8
                            });

                            marker.bindPopup(`
                                <div>
                                    <h4 style="margin: 0 0 5px;"><i class="fas fa-city"></i> ${poblado.nombre}</h4>
                                    <p style="margin: 5px 0;"><strong>Población:</strong> ${poblado.poblacion || 'No disponible'}</p>
                                    <p style="margin: 5px 0;"><strong>Tipo:</strong> ${poblado.tipo || 'Poblado'}</p>
                                </div>
                            `);

                            marker.addTo(pobladosLayer);
                        }
                    });
                } else {
                    console.log('⚠️ No se encontraron poblados o error:', errorPoblados);
                }

                // Cargar vías
                const { data: vias, error: errorVias } = await supabaseClient
                    .from('vias')
                    .select('*')
                    .limit(100); // Limitar para evitar sobrecarga

                if (!errorVias && vias && vias.length > 0) {
                    console.log(`✅ ${vias.length} vías cargadas`);
                    vias.forEach(via => {
                        try {
                            const geom = via.geom || via.geometry;
                            if (geom) {
                                const geoJSON = typeof geom === 'string' ? JSON.parse(geom) : geom;
                                
                                const layer = L.geoJSON(geoJSON, {
                                    style: {
                                        color: '#3498db',
                                        weight: 3,
                                        opacity: 0.7
                                    }
                                });

                                layer.bindPopup(`
                                    <div>
                                        <h4 style="margin: 0 0 5px;"><i class="fas fa-road"></i> ${via.nombre || 'Vía sin nombre'}</h4>
                                        <p style="margin: 5px 0;"><strong>Tipo:</strong> ${via.tipo || 'No especificado'}</p>
                                        <p style="margin: 5px 0;"><strong>Estado:</strong> ${via.estado || 'No disponible'}</p>
                                    </div>
                                `);

                                layer.addTo(viasLayer);
                            }
                        } catch (e) {
                            console.error('Error procesando vía:', e);
                        }
                    });
                } else {
                    console.log('⚠️ No se encontraron vías o error:', errorVias);
                }

                // Cargar zonas urbanas
                const { data: zonas, error: errorZonas } = await supabaseClient
                    .from('zu_ecuador')
                    .select('*')
                    .limit(50); // Limitar para evitar sobrecarga

                if (!errorZonas && zonas && zonas.length > 0) {
                    console.log(`✅ ${zonas.length} zonas urbanas cargadas`);
                    zonas.forEach(zona => {
                        try {
                            const geom = zona.geom || zona.geometry;
                            if (geom) {
                                const geoJSON = typeof geom === 'string' ? JSON.parse(geom) : geom;
                                
                                const layer = L.geoJSON(geoJSON, {
                                    style: {
                                        fillColor: '#f39c12',
                                        fillOpacity: 0.2,
                                        color: '#d68910',
                                        weight: 2
                                    }
                                });

                                layer.bindPopup(`
                                    <div>
                                        <h4 style="margin: 0 0 5px;"><i class="fas fa-building"></i> ${zona.nombre || 'Zona urbana'}</h4>
                                        <p style="margin: 5px 0;"><strong>Área:</strong> ${zona.area || 'No disponible'}</p>
                                        <p style="margin: 5px 0;"><strong>Población:</strong> ${zona.poblacion || 'No disponible'}</p>
                                    </div>
                                `);

                                layer.addTo(zonasLayer);
                            }
                        } catch (e) {
                            console.error('Error procesando zona:', e);
                        }
                    });
                } else {
                    console.log('⚠️ No se encontraron zonas urbanas o error:', errorZonas);
                }
            } catch (error) {
                console.error('❌ Error cargando capas adicionales:', error);
                // Deshabilitar los switches en caso de error
                document.querySelectorAll('.ios-switch input').forEach(input => {
                    input.disabled = true;
                    input.parentElement.style.opacity = '0.5';
                });
            }
        }

        // Actualizar estadísticas
        function actualizarEstadisticas() {
            document.getElementById('totalObservaciones').textContent = observaciones.length;
            
            const especiesUnicas = new Set(observaciones.map(obs => obs.tipo_tortuga));
            document.getElementById('especiesUnicas').textContent = especiesUnicas.size;

            // Actualizar contadores de especies
            const conteoEspecies = {};
            observaciones.forEach(obs => {
                conteoEspecies[obs.tipo_tortuga] = (conteoEspecies[obs.tipo_tortuga] || 0) + 1;
            });

            document.querySelectorAll('.species-btn').forEach(btn => {
                const especie = btn.dataset.species;
                const count = conteoEspecies[especie] || 0;
                btn.querySelector('.count').textContent = count;
            });
        }

        // Toggle sección
        function toggleSection(header) {
            header.classList.toggle('active');
            const content = header.nextElementSibling;
            content.classList.toggle('active');
        }

        // Toggle filtro de especie
        function toggleSpeciesFilter(btn) {
            const especie = btn.dataset.species;
            filtrosActivos[especie] = !filtrosActivos[especie];
            btn.classList.toggle('inactive');
            mostrarObservaciones();
        }

        // Toggle capa
        function toggleLayer(tipo) {
            switch(tipo) {
                case 'poblados':
                    if (map.hasLayer(pobladosLayer)) {
                        map.removeLayer(pobladosLayer);
                    } else {
                        map.addLayer(pobladosLayer);
                    }
                    break;
                case 'vias':
                    if (map.hasLayer(viasLayer)) {
                        map.removeLayer(viasLayer);
                    } else {
                        map.addLayer(viasLayer);
                    }
                    break;
                case 'zonas':
                    if (map.hasLayer(zonasLayer)) {
                        map.removeLayer(zonasLayer);
                    } else {
                        map.addLayer(zonasLayer);
                    }
                    break;
            }
        }

        // Obtener ubicación GPS
        function obtenerUbicacion() {
            if (navigator.geolocation) {
                const btn = event.target.closest('button');
                btn.innerHTML = '<i class="fas fa-location-dot"></i> Obteniendo ubicación... <span class="spinner"></span>';
                btn.disabled = true;

                navigator.geolocation.getCurrentPosition(
                    (position) => {
                        document.getElementById('latitud').value = position.coords.latitude.toFixed(6);
                        document.getElementById('longitud').value = position.coords.longitude.toFixed(6);
                        
                        btn.innerHTML = '<i class="fas fa-location-dot"></i> Obtener mi ubicación';
                        btn.disabled = false;
                        
                        mostrarMensaje('Ubicación obtenida exitosamente', 'success');
                        
                        // Centrar mapa en la ubicación
                        map.setView([position.coords.latitude, position.coords.longitude], 12);
                    },
                    (error) => {
                        btn.innerHTML = '<i class="fas fa-location-dot"></i> Obtener mi ubicación';
                        btn.disabled = false;
                        mostrarMensaje('Error al obtener ubicación: ' + error.message, 'error');
                    }
                );
            } else {
                mostrarMensaje('La geolocalización no está soportada en este navegador', 'error');
            }
        }

        // Mostrar mensaje
        function mostrarMensaje(texto, tipo) {
            const messageEl = document.getElementById('formMessage');
            messageEl.textContent = texto;
            messageEl.className = `message ${tipo}`;
            messageEl.style.display = 'block';
            
            setTimeout(() => {
                messageEl.style.display = 'none';
            }, 5000);
        }

        // Guardar observación
        document.getElementById('observationForm').addEventListener('submit', async (e) => {
            e.preventDefault();
            
            const btn = e.target.querySelector('button[type="submit"]');
            btn.innerHTML = '<i class="fas fa-save"></i> Guardando... <span class="spinner"></span>';
            btn.disabled = true;

            const nuevaObservacion = {
                nombre_observador: document.getElementById('nombreObservador').value,
                tag_id_tortuga: document.getElementById('tagId').value || null,
                tipo_tortuga: document.getElementById('tipoTortuga').value,
                actividad: document.getElementById('actividad').value,
                observaciones: document.getElementById('observaciones').value || null,
                latitud: parseFloat(document.getElementById('latitud').value),
                longitud: parseFloat(document.getElementById('longitud').value)
            };

            try {
                if (isOfflineMode) {
                    // Modo offline
                    nuevaObservacion.id = Date.now();
                    nuevaObservacion.created_at = new Date().toISOString();
                    observaciones.unshift(nuevaObservacion);
                    mostrarMensaje('⚠️ Observación guardada localmente (modo offline)', 'success');
                } else {
                    // Modo online - guardar en Supabase
                    const { data, error } = await supabaseClient
                        .from('observaciones_tortugas')
                        .insert([nuevaObservacion])
                        .select()
                        .single();

                    if (error) throw error;
                    
                    observaciones.unshift(data);
                    mostrarMensaje('✅ Observación guardada exitosamente en Supabase', 'success');
                }

                // Actualizar mapa y estadísticas
                mostrarObservaciones();
                actualizarEstadisticas();
                actualizarListaObservaciones();
                
                // Limpiar formulario
                document.getElementById('observationForm').reset();
                
                // Centrar mapa en nueva observación
                map.setView([nuevaObservacion.latitud, nuevaObservacion.longitud], 12);
                
                // Destacar el nuevo marcador
                setTimeout(() => {
                    focusObservation(nuevaObservacion.id);
                }, 500);
                
            } catch (error) {
                console.error('❌ Error:', error);
                mostrarMensaje('Error al guardar: ' + error.message, 'error');
            } finally {
                btn.innerHTML = '<i class="fas fa-save"></i> Guardar Observación';
                btn.disabled = false;
            }
        });

        // Actualizar datos
        async function actualizarDatos() {
            const btn = event.target.closest('button');
            btn.innerHTML = '<i class="fas fa-sync fa-spin"></i> Actualizando...';
            btn.disabled = true;

            await cargarDatos();
            
            btn.innerHTML = '<i class="fas fa-sync"></i> Actualizar Datos';
            btn.disabled = false;
            
            mostrarMensaje('✅ Datos actualizados correctamente', 'success');
        }

        // Centrar mapa
        function centrarMapa() {
            map.setView([-1.0, -80.7], 8);
        }

        // Inicializar cuando el DOM esté listo
        document.addEventListener('DOMContentLoaded', async () => {
            // Primero inicializar Supabase
            await initSupabase();
            
            // Luego inicializar el mapa
            initMap();
        });
    </script>
</body>
</html>
