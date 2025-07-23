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

        /* Categorías de secciones */
        .category-title {
            font-size: 11px;
            text-transform: uppercase;
            color: #95a5a6;
            font-weight: 600;
            margin: 15px 5px 8px;
            letter-spacing: 0.5px;
        }

        .category-title:first-child {
            margin-top: 0;
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
            cursor: pointer;
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
            grid-template-columns: 1fr 1fr auto;
            gap: 10px;
            align-items: end;
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
            white-space: nowrap;
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
            display: block;
        }

        .message.error {
            background: #fee2e2;
            color: #991b1b;
            border: 1px solid #fca5a5;
            display: block;
        }

        .message.loading {
            background: #cce7ff;
            color: #0066cc;
            border: 1px solid #99d6ff;
            display: block;
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

        .actions-grid {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 10px;
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
                gap: 10px;
            }

            .stats-grid {
                grid-template-columns: 1fr;
            }

            .connection-status {
                top: auto;
                bottom: 20px;
                right: 20px;
            }

            .actions-grid {
                grid-template-columns: 1fr;
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

        <!-- CATEGORÍA: TORTUGAS MARINAS -->
        <div class="category-title">TORTUGAS MARINAS</div>

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
                        <div class="label">Observaciones</div>
                    </div>
                    <div class="stat-card">
                        <div class="value" id="totalSOWT">0</div>
                        <div class="label">Datos SOWT</div>
                    </div>
                    <div class="stat-card">
                        <div class="value" id="especiesUnicas">0</div>
                        <div class="label">Especies Únicas</div>
                    </div>
                    <div class="stat-card">
                        <div class="value" id="ultimaActualizacion">-</div>
                        <div class="label">Última Actualización</div>
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
                        <span class="count" id="count-Verde">0</span>
                    </button>
                    <button class="species-btn species-carey" data-species="Carey" onclick="toggleSpeciesFilter(this)">
                        <span class="indicator"></span>
                        <span>Tortuga Carey</span>
                        <span class="count" id="count-Carey">0</span>
                    </button>
                    <button class="species-btn species-boba" data-species="Boba" onclick="toggleSpeciesFilter(this)">
                        <span class="indicator"></span>
                        <span>Tortuga Boba</span>
                        <span class="count" id="count-Boba">0</span>
                    </button>
                    <button class="species-btn species-laud" data-species="Laúd" onclick="toggleSpeciesFilter(this)">
                        <span class="indicator"></span>
                        <span>Tortuga Laúd</span>
                        <span class="count" id="count-Laúd">0</span>
                    </button>
                    <button class="species-btn species-golfina" data-species="Golfina" onclick="toggleSpeciesFilter(this)">
                        <span class="indicator"></span>
                        <span>Tortuga Golfina</span>
                        <span class="count" id="count-Golfina">0</span>
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
                        <label>Nombre del Observador *</label>
                        <input type="text" class="form-control" id="nombreObservador" required>
                    </div>

                    <div class="form-group">
                        <label>Tag ID de la Tortuga</label>
                        <input type="text" class="form-control" id="tagId" placeholder="Ej: TM-001">
                    </div>

                    <div class="form-group">
                        <label>Tipo de Tortuga *</label>
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
                        <select class="form-control" id="actividad">
                            <option value="">Seleccionar actividad...</option>
                            <option value="Anidación">Anidación</option>
                            <option value="Alimentación">Alimentación</option>
                            <option value="Descanso">Descanso</option>
                            <option value="Migración">Migración</option>
                            <option value="Natación">Natación</option>
                            <option value="Apareamiento">Apareamiento</option>
                            <option value="Emergencia a superficie">Emergencia a superficie</option>
                            <option value="Varada">Varada</option>
                            <option value="Otro">Otro</option>
                        </select>
                    </div>

                    <div class="form-group">
                        <label>Observaciones</label>
                        <textarea class="form-control" id="observaciones" placeholder="Detalles adicionales..."></textarea>
                    </div>

                    <div class="form-group">
                        <label>Coordenadas *</label>
                        <div class="coords-group">
                            <input type="number" class="form-control" id="latitud" placeholder="Latitud" step="0.000001" min="-90" max="90" required>
                            <input type="number" class="form-control" id="longitud" placeholder="Longitud" step="0.000001" min="-180" max="180" required>
                            <button type="button" class="btn btn-gps" onclick="obtenerUbicacion()">
                                <i class="fas fa-location-dot"></i> GPS
                            </button>
                        </div>
                    </div>

                    <div id="formMessage" class="message"></div>

                    <button type="submit" class="btn btn-primary" style="margin-top: 16px;">
                        <i class="fas fa-save"></i> Guardar Observación
                    </button>
                </form>
            </div>
        </div>

        <!-- CATEGORÍA: CAPAS GEOGRÁFICAS -->
        <div class="category-title">CAPAS GEOGRÁFICAS</div>

        <!-- Sección de Capas Base -->
        <div class="section">
            <div class="section-header" onclick="toggleSection(this)">
                <div class="section-header-content">
                    <i class="fas fa-map"></i>
                    <span>Capas Base</span>
                </div>
                <i class="fas fa-chevron-down arrow"></i>
            </div>
            <div class="section-content">
                <div class="layer-controls">
                    <div class="layer-item">
                        <label for="togglePoblados"><i class="fas fa-city"></i> Poblados</label>
                        <div class="ios-switch">
                            <input type="checkbox" id="togglePoblados" onchange="toggleLayer('poblados', this.checked)">
                            <div class="ios-switch-bg"></div>
                            <div class="slider"></div>
                        </div>
                    </div>
                    <div class="layer-item">
                        <label for="toggleVias"><i class="fas fa-road"></i> Vías</label>
                        <div class="ios-switch">
                            <input type="checkbox" id="toggleVias" onchange="toggleLayer('vias', this.checked)">
                            <div class="ios-switch-bg"></div>
                            <div class="slider"></div>
                        </div>
                    </div>
                    <div class="layer-item">
                        <label for="toggleZonas"><i class="fas fa-building"></i> Zonas Urbanas</label>
                        <div class="ios-switch">
                            <input type="checkbox" id="toggleZonas" onchange="toggleLayer('zonas', this.checked)">
                            <div class="ios-switch-bg"></div>
                            <div class="slider"></div>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <!-- Sección de Datos Climáticos -->
        <div class="section">
            <div class="section-header" onclick="toggleSection(this)">
                <div class="section-header-content">
                    <i class="fas fa-cloud-rain"></i>
                    <span>Datos Climáticos</span>
                </div>
                <i class="fas fa-chevron-down arrow"></i>
            </div>
            <div class="section-content">
                <div class="layer-controls">
                    <div class="layer-item">
                        <label for="togglePrecip2018"><i class="fas fa-tint"></i> Precipitación 2018</label>
                        <div class="ios-switch">
                            <input type="checkbox" id="togglePrecip2018" onchange="toggleLayer('precip2018', this.checked)">
                            <div class="ios-switch-bg"></div>
                            <div class="slider"></div>
                        </div>
                    </div>
                    <div class="layer-item">
                        <label for="togglePrecip2019"><i class="fas fa-tint"></i> Precipitación 2019</label>
                        <div class="ios-switch">
                            <input type="checkbox" id="togglePrecip2019" onchange="toggleLayer('precip2019', this.checked)">
                            <div class="ios-switch-bg"></div>
                            <div class="slider"></div>
                        </div>
                    </div>
                    <div class="layer-item">
                        <label for="toggleEstaciones"><i class="fas fa-broadcast-tower"></i> Estaciones Meteorológicas</label>
                        <div class="ios-switch">
                            <input type="checkbox" id="toggleEstaciones" onchange="toggleLayer('estaciones', this.checked)">
                            <div class="ios-switch-bg"></div>
                            <div class="slider"></div>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <!-- CATEGORÍA: HERRAMIENTAS -->
        <div class="category-title">HERRAMIENTAS</div>

        <!-- Sección de Acciones -->
        <div class="section">
            <div class="section-header" onclick="toggleSection(this)">
                <div class="section-header-content">
                    <i class="fas fa-tools"></i>
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
        // Configuración de Supabase
        const SUPABASE_URL = 'https://dhvlvfhhagicnvfekxgc.supabase.co';
        const SUPABASE_ANON_KEY = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImRodmx2ZmhoYWdpY252ZmVreGdjIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NTAyNzYwNjIsImV4cCI6MjA2NTg1MjA2Mn0.U29qJQeU41AZkJh7Xm6Fj7J4wBo-UK_nqkhhtIpWCNQ';
        
        // Variables globales
        let supabaseClient = null;
        let isOfflineMode = false;
        let map;
        let observacionesLayer;
        let sowtLayer;
        let pobladosLayer;
        let viasLayer;
        let zonasLayer;
        let precip2018Layer;
        let precip2019Layer;
        let estacionesLayer;
        let observaciones = [];
        let sowtData = [];
        let marcadores = {};
        let marcadoresSOWT = {};
        let marcadorActivo = null;
        let filtrosActivos = {
            Verde: true,
            Carey: true,
            Boba: true,
            Laúd: true,
            Golfina: true
        };

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
                nombre_observador: 'Ana Ruiz',
                tag_id_tortuga: 'TM-003',
                tipo_tortuga: 'Boba',
                actividad: 'Migración',
                observaciones: 'Tortuga joven en ruta migratoria',
                latitud: -0.6267,
                longitud: -80.4123,
                created_at: new Date().toISOString()
            },
            {
                id: 4,
                nombre_observador: 'Jorge Silva',
                tag_id_tortuga: 'TM-004',
                tipo_tortuga: 'Laúd',
                actividad: 'Alimentación',
                observaciones: 'Tortuga adulta alimentándose de medusas',
                latitud: -2.1962,
                longitud: -80.8887,
                created_at: new Date().toISOString()
            },
            {
                id: 5,
                nombre_observador: 'Paola Castro',
                tag_id_tortuga: 'TM-005',
                tipo_tortuga: 'Golfina',
                actividad: 'Anidación',
                observaciones: 'Arribada observada en playa protegida',
                latitud: -1.5433,
                longitud: -80.9678,
                created_at: new Date().toISOString()
            }
        ];

        // Datos SOWT simulados
        const sowtDataExample = [
            {
                id: 'sowt-1',
                species: 'Green turtle',
                status: 'Alive',
                size: 'Adult',
                sex: 'Female',
                latitude: -0.8,
                longitude: -80.6,
                observation_date: '2024-07-15',
                location: 'Puerto López'
            },
            {
                id: 'sowt-2',
                species: 'Hawksbill turtle',
                status: 'Alive',
                size: 'Juvenile',
                sex: 'Unknown',
                latitude: -1.1,
                longitude: -80.7,
                observation_date: '2024-07-10',
                location: 'Manta Bay'
            },
            {
                id: 'sowt-3',
                species: 'Loggerhead turtle',
                status: 'Alive',
                size: 'Adult',
                sex: 'Male',
                latitude: -1.8,
                longitude: -80.9,
                observation_date: '2024-07-12',
                location: 'Salinas'
            }
        ];

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
                
                if (error && error.code !== 'PGRST116') throw error; // PGRST116 = tabla vacía
                
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
            sowtLayer = L.layerGroup().addTo(map);
            pobladosLayer = L.layerGroup();
            viasLayer = L.layerGroup();
            zonasLayer = L.layerGroup();
            precip2018Layer = L.layerGroup();
            precip2019Layer = L.layerGroup();
            estacionesLayer = L.layerGroup();

            // Cargar todas las capas con datos fijos
            setupAllLayers();
            cargarDatos();
        }

        // Configurar todas las capas con datos fijos
        function setupAllLayers() {
            setupClimateLayers();
            setupInfrastructureLayers();
        }

        // Cargar todos los datos
        async function cargarDatos() {
            console.log('🔄 Iniciando carga de datos...');
            
            await cargarObservaciones();
            await cargarSOWTData();
            
            actualizarEstadisticas();
            actualizarListaObservaciones();
            
            console.log('✅ Carga de datos completada');
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

                    if (error && error.code !== 'PGRST116') throw error;
                    
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
                mostrarObservaciones();
            }
        }

        // Cargar datos SOWT
        async function cargarSOWTData() {
            try {
                if (isOfflineMode) {
                    sowtData = sowtDataExample;
                    console.log('📱 Usando datos SOWT de ejemplo (modo offline)');
                } else {
                    const { data, error } = await supabaseClient
                        .from('otm_ec_sowt')
                        .select('*')
                        .limit(1000);

                    if (error && error.code !== 'PGRST116') {
                        console.log('⚠️ Tabla SOWT no disponible, usando datos de ejemplo');
                        sowtData = sowtDataExample;
                    } else {
                        sowtData = data || sowtDataExample;
                        console.log(`✅ ${sowtData.length} registros SOWT cargados`);
                        
                        if (sowtData.length === 0) {
                            sowtData = sowtDataExample;
                        }
                    }
                }
                
                mostrarSOWTData();
            } catch (error) {
                console.error('❌ Error cargando datos SOWT:', error);
                sowtData = sowtDataExample;
                mostrarSOWTData();
            }
        }

        // Crear icono triangular personalizado
        function createTriangleIcon(color, size = 20) {
            return L.divIcon({
                html: `<div style="
                    width: 0; 
                    height: 0; 
                    border-left: ${size/2}px solid transparent;
                    border-right: ${size/2}px solid transparent;
                    border-bottom: ${size}px solid ${color};
                    transform: rotate(0deg);
                "></div>`,
                className: 'triangle-icon',
                iconSize: [size, size],
                iconAnchor: [size/2, size]
            });
        }

        // Mostrar datos SOWT en el mapa
        function mostrarSOWTData() {
            sowtLayer.clearLayers();
            marcadoresSOWT = {};

            sowtData.forEach(registro => {
                // Mapear especies comunes de SOWT a nuestro sistema
                let especie = 'Verde'; // Por defecto
                if (registro.species) {
                    const sp = registro.species.toLowerCase();
                    if (sp.includes('green') || sp.includes('verde')) especie = 'Verde';
                    else if (sp.includes('hawksbill') || sp.includes('carey')) especie = 'Carey';
                    else if (sp.includes('loggerhead') || sp.includes('boba')) especie = 'Boba';
                    else if (sp.includes('leatherback') || sp.includes('laud')) especie = 'Laúd';
                    else if (sp.includes('olive') || sp.includes('golfina')) especie = 'Golfina';
                }

                if (filtrosActivos[especie] && registro.latitude && registro.longitude) {
                    const marker = L.marker([registro.latitude, registro.longitude], {
                        icon: createTriangleIcon(coloresEspecies[especie], 16)
                    });

                    const fecha = registro.observation_date || registro.date || 'No especificada';

                    marker.bindPopup(`
                        <div style="min-width: 280px;">
                            <h4 style="margin: 0 0 10px; color: ${coloresEspecies[especie]};">
                                🔺 SOWT - ${especie}
                            </h4>
                            <p style="margin: 5px 0;"><strong>Especie:</strong> ${registro.species || 'No especificada'}</p>
                            <p style="margin: 5px 0;"><strong>Estado:</strong> ${registro.status || 'No especificado'}</p>
                            <p style="margin: 5px 0;"><strong>Tamaño:</strong> ${registro.size || 'No especificado'}</p>
                            <p style="margin: 5px 0;"><strong>Sexo:</strong> ${registro.sex || 'No especificado'}</p>
                            <p style="margin: 5px 0;"><strong>Fecha:</strong> ${fecha}</p>
                            <p style="margin: 5px 0;"><strong>Ubicación:</strong> ${registro.location || 'No especificada'}</p>
                            <p style="margin: 5px 0;"><strong>Coordenadas:</strong> ${registro.latitude.toFixed(6)}, ${registro.longitude.toFixed(6)}</p>
                            <p style="margin: 5px 0; font-size: 11px; color: #666;">Fuente: SOWT Database</p>
                        </div>
                    `);

                    marker.addTo(sowtLayer);
                    marcadoresSOWT[registro.id] = marker;
                }
            });
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
                                🐢 ${obs.tipo_tortuga}
                            </h4>
                            <p style="margin: 5px 0;"><strong>Observador:</strong> ${obs.nombre_observador}</p>
                            <p style="margin: 5px 0;"><strong>Tag ID:</strong> ${obs.tag_id_tortuga || 'No registrado'}</p>
                            <p style="margin: 5px 0;"><strong>Actividad:</strong> ${obs.actividad || 'No especificada'}</p>
                            <p style="margin: 5px 0;"><strong>Observaciones:</strong> ${obs.observaciones || 'Sin observaciones'}</p>
                            <p style="margin: 5px 0;"><strong>Fecha:</strong> ${fecha} ${hora}</p>
                            <p style="margin: 5px 0;"><strong>Coordenadas:</strong> ${obs.latitud.toFixed(6)}, ${obs.longitud.toFixed(6)}</p>
                            <p style="margin: 5px 0; font-size: 11px; color: #666;">Fuente: Observación Local</p>
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
                    <p><strong>Actividad:</strong> ${obs.actividad || 'No especificada'}</p>
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
            }
        }

        // Configurar capas climáticas con datos fijos
        function setupClimateLayers() {
            // Datos de precipitación 2018 (fijos)
            const precipitationData2018 = [
                { coords: [[-0.5, -81.2], [-0.2, -81.2], [-0.2, -80.8], [-0.5, -80.8]], value: 'Alta', mm: 2500 },
                { coords: [[-1.5, -81.0], [-1.2, -81.0], [-1.2, -80.6], [-1.5, -80.6]], value: 'Media', mm: 1800 },
                { coords: [[-0.8, -80.8], [-0.5, -80.8], [-0.5, -80.4], [-0.8, -80.4]], value: 'Baja', mm: 1200 },
                { coords: [[-2.5, -81.5], [-2.0, -81.5], [-2.0, -80.8], [-2.5, -80.8]], value: 'Media', mm: 1600 }
            ];

            precipitationData2018.forEach(data => {
                const color = data.value === 'Alta' ? '#2980b9' : data.value === 'Media' ? '#3498db' : '#85c1e9';
                L.polygon(data.coords, {
                    color: color,
                    fillColor: color,
                    fillOpacity: 0.4,
                    weight: 2
                }).bindPopup(`
                    <h4><i class="fas fa-tint"></i> Precipitación 2018</h4>
                    <p><strong>Nivel:</strong> ${data.value}</p>
                    <p><strong>Precipitación:</strong> ${data.mm} mm</p>
                    <p><strong>Estación:</strong> Seca/Lluviosa</p>
                `).addTo(precip2018Layer);
            });

            // Datos de precipitación 2019 (fijos)
            const precipitationData2019 = [
                { coords: [[-0.7, -81.0], [-0.4, -81.0], [-0.4, -80.6], [-0.7, -80.6]], value: 'Alta', mm: 2700 },
                { coords: [[-1.3, -80.8], [-1.0, -80.8], [-1.0, -80.4], [-1.3, -80.4]], value: 'Media', mm: 1900 },
                { coords: [[-0.6, -80.6], [-0.3, -80.6], [-0.3, -80.2], [-0.6, -80.2]], value: 'Baja', mm: 1100 },
                { coords: [[-2.3, -81.2], [-2.0, -81.2], [-2.0, -80.9], [-2.3, -80.9]], value: 'Alta', mm: 2400 }
            ];

            precipitationData2019.forEach(data => {
                const color = data.value === 'Alta' ? '#1e3a8a' : data.value === 'Media' ? '#1e40af' : '#60a5fa';
                L.polygon(data.coords, {
                    color: color,
                    fillColor: color,
                    fillOpacity: 0.4,
                    weight: 2
                }).bindPopup(`
                    <h4><i class="fas fa-tint"></i> Precipitación 2019</h4>
                    <p><strong>Nivel:</strong> ${data.value}</p>
                    <p><strong>Precipitación:</strong> ${data.mm} mm</p>
                    <p><strong>Variación:</strong> ${data.mm > 2000 ? '+15%' : data.mm > 1500 ? '+5%' : '-10%'} vs 2018</p>
                `).addTo(precip2019Layer);
            });

            // Estaciones meteorológicas (fijas)
            const stations = [
                { coords: [-0.9, -80.7], name: 'Estación Manta', tipo: 'Automática', altitud: 15, institucion: 'INAMHI' },
                { coords: [-1.0, -80.6], name: 'Estación Montecristi', tipo: 'Convencional', altitud: 45, institucion: 'INAMHI' },
                { coords: [-0.5, -80.9], name: 'Estación Puerto López', tipo: 'Automática', altitud: 8, institucion: 'Parque Nacional' },
                { coords: [-1.3, -80.4], name: 'Estación Jipijapa', tipo: 'Convencional', altitud: 180, institucion: 'INAMHI' },
                { coords: [-2.2, -80.9], name: 'Estación Salinas', tipo: 'Automática', altitud: 5, institucion: 'INOCAR' },
                { coords: [-0.4, -80.4], name: 'Estación Bahía de Caráquez', tipo: 'Pluviométrica', altitud: 12, institucion: 'INAMHI' }
            ];

            stations.forEach(station => {
                L.marker(station.coords, {
                    icon: L.divIcon({
                        className: 'weather-station-icon',
                        html: '<i class="fas fa-broadcast-tower" style="color: #8b4513; font-size: 18px;"></i>',
                        iconSize: [20, 20]
                    })
                }).bindPopup(`
                    <h4><i class="fas fa-broadcast-tower"></i> ${station.name}</h4>
                    <p><strong>Tipo:</strong> ${station.tipo}</p>
                    <p><strong>Altitud:</strong> ${station.altitud} msnm</p>
                    <p><strong>Institución:</strong> ${station.institucion}</p>
                    <p><strong>Estado:</strong> Activa</p>
                    <p><strong>Datos:</strong> Temperatura, Humedad, Precipitación, Viento</p>
                `).addTo(estacionesLayer);
            });
        }

        // Configurar capas de infraestructura con datos fijos
        function setupInfrastructureLayers() {
            // Poblados costeros del Ecuador (datos fijos ampliados)
            const poblados = [
                { coords: [-0.9553, -80.7339], name: 'Manta', poblacion: 264281, provincia: 'Manabí', tipo: 'Ciudad Principal', puerto: true },
                { coords: [-1.2642, -80.8118], name: 'Portoviejo', poblacion: 321010, provincia: 'Manabí', tipo: 'Capital Provincial', puerto: false },
                { coords: [-0.3708, -80.4056], name: 'Bahía de Caráquez', poblacion: 27316, provincia: 'Manabí', tipo: 'Ciudad Turística', puerto: true },
                { coords: [-1.0180, -80.7055], name: 'Montecristi', poblacion: 73070, provincia: 'Manabí', tipo: 'Ciudad Histórica', puerto: false },
                { coords: [-1.5433, -80.9678], name: 'Jipijapa', poblacion: 51475, provincia: 'Manabí', tipo: 'Ciudad Comercial', puerto: false },
                { coords: [-0.6267, -80.4123], name: 'Puerto López', poblacion: 20451, provincia: 'Manabí', tipo: 'Pueblo Pesquero', puerto: true },
                { coords: [-2.1962, -80.8887], name: 'Salinas', poblacion: 68675, provincia: 'Santa Elena', tipo: 'Balneario', puerto: true },
                { coords: [-2.2108, -80.9711], name: 'La Libertad', poblacion: 115617, provincia: 'Santa Elena', tipo: 'Puerto Pesquero', puerto: true },
                { coords: [-0.9667, -80.5833], name: 'Jaramijó', poblacion: 18486, provincia: 'Manabí', tipo: 'Pueblo Costero', puerto: true },
                { coords: [-0.8333, -80.4167], name: 'San Vicente', poblacion: 19116, provincia: 'Manabí', tipo: 'Pueblo Agrícola', puerto: false }
            ];

            poblados.forEach(poblado => {
                const radius = Math.max(6, Math.min(poblado.poblacion / 20000, 15));
                const color = poblado.puerto ? '#3498db' : '#8e44ad';
                
                const marker = L.circleMarker(poblado.coords, {
                    radius: radius,
                    fillColor: color,
                    color: '#fff',
                    weight: 2,
                    opacity: 1,
                    fillOpacity: 0.7
                });

                marker.bindPopup(`
                    <h4><i class="fas fa-city"></i> ${poblado.name}</h4>
                    <p><strong>Población:</strong> ${poblado.poblacion.toLocaleString()}</p>
                    <p><strong>Provincia:</strong> ${poblado.provincia}</p>
                    <p><strong>Tipo:</strong> ${poblado.tipo}</p>
                    <p><strong>Puerto:</strong> ${poblado.puerto ? 'Sí' : 'No'}</p>
