# planety
planety
function interaktivni_planetarni_system()
    close all; clc;

    % ==========================================================
    % INTERAKTIVNÍ PLANETÁRNÍ SOUSTAVA - OCTAVE
    % ==========================================================
    % Co simulace ukazuje:
    % - oběh planet kolem Slunce
    % - eliptické dráhy při změně počáteční rychlosti
    % - kometu / gravitační prak
    % - Měsíc obíhající kolem Země
    % - Saturn s prstencem
    % - hvězdné pozadí a stopy drah
    %
    % Jednotky:
    % vzdálenost = AU
    % čas = roky
    % hmotnost = násobky hmotnosti Slunce
    %
    % Poznámka:
    % Velikosti planet nejsou realistické v měřítku, jsou zvětšené,
    % aby byly dobře vidět.
    % ==========================================================

    % Fyzikální konstanta v jednotkách AU, rok, M_sun
    G = 4*pi^2;

    % ----------------------------------------------------------
    % Vytvoření okna
    % ----------------------------------------------------------
    fig = figure( ...
        'Name', 'Interaktivní planetární soustava', ...
        'NumberTitle', 'off', ...
        'Color', 'k', ...
        'Position', [60 60 1300 760]);

    ax = axes( ...
        'Parent', fig, ...
        'Position', [0.05 0.08 0.68 0.87], ...
        'Color', 'k', ...
        'XColor', 'w', ...
        'YColor', 'w');
    hold(ax, 'on');
    axis(ax, 'equal');
    axis(ax, [-15 15 -15 15]);
    grid(ax, 'on');
    xlabel(ax, 'x [AU]', 'Color', 'w');
    ylabel(ax, 'y [AU]', 'Color', 'w');
    title(ax, 'Planetární soustava', 'Color', 'w', 'FontSize', 16);

    % ----------------------------------------------------------
    % Hvězdné pozadí
    % ----------------------------------------------------------
    rng(1); % aby byly hvězdy vždy stejné
    nStars = 300;
    sx = -15 + 30*rand(nStars,1);
    sy = -15 + 30*rand(nStars,1);
    ss = 4 + 10*rand(nStars,1);
    for i = 1:nStars
        plot(ax, sx(i), sy(i), '.', 'Color', [1 1 1]*rand()*0.8 + 0.2, 'MarkerSize', ss(i));
    end

    % ----------------------------------------------------------
    % Ovládací panel
    % ----------------------------------------------------------
    panel = uipanel( ...
        'Parent', fig, ...
        'Title', 'Ovládání simulace', ...
        'BackgroundColor', [0.12 0.12 0.12], ...
        'ForegroundColor', 'w', ...
        'Position', [0.76 0.08 0.22 0.87]);

    % Text s instrukcí
    uicontrol(panel, ...
        'Style', 'text', ...
        'Units', 'normalized', ...
        'Position', [0.07 0.90 0.86 0.07], ...
        'String', sprintf(['Posuň posuvník.\n' ...
                           'Simulace se automaticky\n' ...
                           'spustí znovu s novými\n' ...
                           'počátečními podmínkami.']), ...
        'BackgroundColor', [0.12 0.12 0.12], ...
        'ForegroundColor', 'w', ...
        'FontSize', 11);

    % 1) rychlost Země
    txtEarth = uicontrol(panel, ...
        'Style', 'text', ...
        'Units', 'normalized', ...
        'Position', [0.08 0.79 0.84 0.05], ...
        'String', 'Rychlost Země: 1.00×', ...
        'BackgroundColor', [0.12 0.12 0.12], ...
        'ForegroundColor', [0.7 0.85 1], ...
        'FontSize', 11);

    sEarth = uicontrol(panel, ...
        'Style', 'slider', ...
        'Units', 'normalized', ...
        'Position', [0.08 0.75 0.84 0.05], ...
        'Min', 0.80, 'Max', 1.20, 'Value', 1.00, ...
        'Callback', @(src,evt) slider_changed(src, fig, 'earth'));

    % 2) hmotnost Slunce
    txtSun = uicontrol(panel, ...
        'Style', 'text', ...
        'Units', 'normalized', ...
        'Position', [0.08 0.65 0.84 0.05], ...
        'String', 'Hmotnost Slunce: 1.00×', ...
        'BackgroundColor', [0.12 0.12 0.12], ...
        'ForegroundColor', [1 0.9 0.4], ...
        'FontSize', 11);

    sSun = uicontrol(panel, ...
        'Style', 'slider', ...
        'Units', 'normalized', ...
        'Position', [0.08 0.61 0.84 0.05], ...
        'Min', 0.70, 'Max', 1.30, 'Value', 1.00, ...
        'Callback', @(src,evt) slider_changed(src, fig, 'sun'));

    % 3) rychlost komety
    txtCometV = uicontrol(panel, ...
        'Style', 'text', ...
        'Units', 'normalized', ...
        'Position', [0.08 0.51 0.84 0.05], ...
        'String', 'Rychlost komety: 5.50 AU/rok', ...
        'BackgroundColor', [0.12 0.12 0.12], ...
        'ForegroundColor', [0.7 1 1], ...
        'FontSize', 11);

    sCometV = uicontrol(panel, ...
        'Style', 'slider', ...
        'Units', 'normalized', ...
        'Position', [0.08 0.47 0.84 0.05], ...
        'Min', 3.0, 'Max', 8.5, 'Value', 5.5, ...
        'Callback', @(src,evt) slider_changed(src, fig, 'cometv'));

    % 4) úhel komety
    txtCometA = uicontrol(panel, ...
        'Style', 'text', ...
        'Units', 'normalized', ...
        'Position', [0.08 0.37 0.84 0.05], ...
        'String', 'Úhel komety: -8.0°', ...
        'BackgroundColor', [0.12 0.12 0.12], ...
        'ForegroundColor', [0.7 1 1], ...
        'FontSize', 11);

    sCometA = uicontrol(panel, ...
        'Style', 'slider', ...
        'Units', 'normalized', ...
        'Position', [0.08 0.33 0.84 0.05], ...
        'Min', -25, 'Max', 25, 'Value', -8, ...
        'Callback', @(src,evt) slider_changed(src, fig, 'cometa'));

    % tlačítko pauza
    btnPause = uicontrol(panel, ...
        'Style', 'togglebutton', ...
        'Units', 'normalized', ...
        'Position', [0.08 0.20 0.38 0.07], ...
        'String', 'Pauza', ...
        'FontSize', 11, ...
        'Callback', @(src,evt) toggle_pause(src, fig));

    % tlačítko reset
    btnReset = uicontrol(panel, ...
        'Style', 'pushbutton', ...
        'Units', 'normalized', ...
        'Position', [0.54 0.20 0.38 0.07], ...
        'String', 'Reset', ...
        'FontSize', 11, ...
        'Callback', @(src,evt) reset_sim(fig));

    % informační text
    txtInfo = uicontrol(panel, ...
        'Style', 'text', ...
        'Units', 'normalized', ...
        'Position', [0.07 0.04 0.86 0.11], ...
        'String', sprintf(['Didaktický tip:\n' ...
                           '- menší rychlost Země -> pád ke Slunci\n' ...
                           '- větší rychlost Země -> protáhlá elipsa\n' ...
                           '- změna komety -> průlet nebo gravitační prak']), ...
        'BackgroundColor', [0.12 0.12 0.12], ...
        'ForegroundColor', 'w', ...
        'FontSize', 10);

    % ----------------------------------------------------------
    % Struktura aplikace
    % ----------------------------------------------------------
    app = struct();
    app.fig = fig;
    app.ax = ax;
    app.G = G;
    app.dt = 0.0012;
    app.time = 0;
    app.trail_len = 1200;
    app.changed = true;
    app.paused = false;

    app.sEarth = sEarth;
    app.sSun = sSun;
    app.sCometV = sCometV;
    app.sCometA = sCometA;

    app.txtEarth = txtEarth;
    app.txtSun = txtSun;
    app.txtCometV = txtCometV;
    app.txtCometA = txtCometA;
    app.btnPause = btnPause;

    app.sunPlot = [];
    app.bodyPlots = [];
    app.trailPlots = [];
    app.textPlots = [];
    app.ringPlot = [];
    app.bodies = [];

    guidata(fig, app);

    % ----------------------------------------------------------
    % Hlavní smyčka
    % ----------------------------------------------------------
    while ishandle(fig)
        app = guidata(fig);

        if app.changed
            app = initialize_simulation(app);
            guidata(fig, app);
        end

        if ~app.paused
            app = step_simulation(app);
            app = redraw_scene(app);
            guidata(fig, app);
        end

        drawnow();
    end
end

% ==============================================================
% CALLBACKY
% ==============================================================

function slider_changed(src, fig, what)
    app = guidata(fig);
    val = get(src, 'Value');

    switch what
        case 'earth'
            set(app.txtEarth, 'String', sprintf('Rychlost Země: %.2f×', val));
        case 'sun'
            set(app.txtSun, 'String', sprintf('Hmotnost Slunce: %.2f×', val));
        case 'cometv'
            set(app.txtCometV, 'String', sprintf('Rychlost komety: %.2f AU/rok', val));
        case 'cometa'
            set(app.txtCometA, 'String', sprintf('Úhel komety: %.1f°', val));
    end

    app.changed = true;
    guidata(fig, app);
end

function toggle_pause(src, fig)
    app = guidata(fig);
    app.paused = get(src, 'Value');

    if app.paused
        set(src, 'String', 'Pokračovat');
    else
        set(src, 'String', 'Pauza');
    end

    guidata(fig, app);
end

function reset_sim(fig)
    app = guidata(fig);
    app.changed = true;
    guidata(fig, app);
end

% ==============================================================
% INICIALIZACE SIMULACE
% ==============================================================

function app = initialize_simulation(app)
    ax = app.ax;

    % Smazání starých objektů kromě hvězd
    delete(findall(ax, 'Tag', 'simobj'));

    app.time = 0;

    % Parametry ze sliderů
    earth_scale = get(app.sEarth, 'Value');
    sun_scale   = get(app.sSun, 'Value');
    comet_speed = get(app.sCometV, 'Value');
    comet_angle = deg2rad(get(app.sCometA, 'Value'));

    app.Msun = 1.0 * sun_scale;

    % ----------------------------------------------------------
    % Definice těles
    % ----------------------------------------------------------
    % Hmotnosti v násobcích hmotnosti Slunce
    M_earth   = 3.00e-6;
    M_mars    = 3.23e-7;
    M_jupiter = 9.54e-4;
    M_saturn  = 2.86e-4;
    M_moon    = 3.70e-8;
    M_comet   = 1e-14;   % zanedbatelná hmotnost

    % Pomocná funkce pro kruhovou rychlost kolem Slunce
    vcirc = @(r) sqrt(app.G * app.Msun / r);

    % Fáze planet
    aE = deg2rad(35);
    aM = deg2rad(155);
    aJ = deg2rad(235);
    aS = deg2rad(300);

    % Země
    bodies(1).name = 'Země';
    bodies(1).mass = M_earth;
    bodies(1).color = [0.2 0.55 1.0];
    bodies(1).size = 9;
    bodies(1).pos = [1.0*cos(aE), 1.0*sin(aE)];
    vE = earth_scale * vcirc(1.0);
    bodies(1).vel = [-vE*sin(aE), vE*cos(aE)];
    bodies(1).trailx = bodies(1).pos(1);
    bodies(1).traily = bodies(1).pos(2);

    % Mars
    bodies(2).name = 'Mars';
    bodies(2).mass = M_mars;
    bodies(2).color = [1.0 0.35 0.25];
    bodies(2).size = 7;
    bodies(2).pos = [1.52*cos(aM), 1.52*sin(aM)];
    vM = vcirc(1.52);
    bodies(2).vel = [-vM*sin(aM), vM*cos(aM)];
    bodies(2).trailx = bodies(2).pos(1);
    bodies(2).traily = bodies(2).pos(2);

    % Jupiter
    bodies(3).name = 'Jupiter';
    bodies(3).mass = M_jupiter;
    bodies(3).color = [0.92 0.78 0.58];
    bodies(3).size = 13;
    bodies(3).pos = [5.20*cos(aJ), 5.20*sin(aJ)];
    vJ = vcirc(5.20);
    bodies(3).vel = [-vJ*sin(aJ), vJ*cos(aJ)];
    bodies(3).trailx = bodies(3).pos(1);
    bodies(3).traily = bodies(3).pos(2);

    % Saturn
    bodies(4).name = 'Saturn';
    bodies(4).mass = M_saturn;
    bodies(4).color = [0.93 0.85 0.55];
    bodies(4).size = 12;
    bodies(4).pos = [9.58*cos(aS), 9.58*sin(aS)];
    vS = vcirc(9.58);
    bodies(4).vel = [-vS*sin(aS), vS*cos(aS)];
    bodies(4).trailx = bodies(4).pos(1);
    bodies(4).traily = bodies(4).pos(2);

    % Měsíc kolem Země
    % vzdálenost cca 0.00257 AU
    moon_r = 0.00257;
    moon_v_rel = sqrt(app.G * M_earth / moon_r);

    earth_pos = bodies(1).pos;
    earth_vel = bodies(1).vel;

    bodies(5).name = 'Měsíc';
    bodies(5).mass = M_moon;
    bodies(5).color = [0.9 0.9 0.9];
    bodies(5).size = 5;
    bodies(5).pos = earth_pos + [moon_r, 0];
    bodies(5).vel = earth_vel + [0, moon_v_rel];
    bodies(5).trailx = bodies(5).pos(1);
    bodies(5).traily = bodies(5).pos(2);

    % Kometa
    bodies(6).name = 'Kometa';
    bodies(6).mass = M_comet;
    bodies(6).color = [0.6 1.0 1.0];
    bodies(6).size = 6;
    bodies(6).pos = [-13, 3.0];
    bodies(6).vel = comet_speed * [cos(comet_angle), sin(comet_angle)];
    bodies(6).trailx = bodies(6).pos(1);
    bodies(6).traily = bodies(6).pos(2);

    app.bodies = bodies;

    % ----------------------------------------------------------
    % Grafické objekty
    % ----------------------------------------------------------
    app.sunPlot = plot(ax, 0, 0, 'o', ...
        'MarkerSize', 18, ...
        'MarkerFaceColor', [1 0.9 0.1], ...
        'MarkerEdgeColor', [1 0.95 0.4], ...
        'Tag', 'simobj');

    n = numel(app.bodies);
    app.trailPlots = gobjects(1, n);
    app.bodyPlots  = gobjects(1, n);
    app.textPlots  = gobjects(1, n);

    for i = 1:n
        app.trailPlots(i) = plot(ax, app.bodies(i).trailx, app.bodies(i).traily, '-', ...
            'Color', app.bodies(i).color, 'LineWidth', 1.2, 'Tag', 'simobj');

        app.bodyPlots(i) = plot(ax, app.bodies(i).pos(1), app.bodies(i).pos(2), 'o', ...
            'MarkerSize', app.bodies(i).size, ...
            'MarkerFaceColor', app.bodies(i).color, ...
            'MarkerEdgeColor', app.bodies(i).color, ...
            'Tag', 'simobj');

        app.textPlots(i) = text(ax, ...
            app.bodies(i).pos(1) + 0.18, ...
            app.bodies(i).pos(2) + 0.18, ...
            app.bodies(i).name, ...
            'Color', app.bodies(i).color, ...
            'FontSize', 9, ...
            'Tag', 'simobj');
    end

    % Saturnův prstenec - pouze dekorativně
    th = linspace(0, 2*pi, 200);
    sat = app.bodies(4).pos;
    ring_x = sat(1) + 0.80*cos(th);
    ring_y = sat(2) + 0.28*sin(th);
    app.ringPlot = plot(ax, ring_x, ring_y, '-', ...
        'Color', [0.95 0.9 0.75], ...
        'LineWidth', 1.0, ...
        'Tag', 'simobj');

    app.changed = false;
end

% ==============================================================
% JEDEN KROK SIMULACE
% ==============================================================

function app = step_simulation(app)
    n = numel(app.bodies);

    pos = zeros(n, 2);
    vel = zeros(n, 2);
    mass = zeros(n, 1);

    for i = 1:n
        pos(i,:) = app.bodies(i).pos;
        vel(i,:) = app.bodies(i).vel;
        mass(i) = app.bodies(i).mass;
    end

    % Leapfrog / velocity Verlet - stabilnější než obyčejný Euler
    acc1 = compute_acc(pos, mass, app.Msun, app.G);

    vel_half = vel + 0.5 * app.dt * acc1;
    pos_new  = pos + app.dt * vel_half;

    acc2 = compute_acc(pos_new, mass, app.Msun, app.G);
    vel_new = vel_half + 0.5 * app.dt * acc2;

    % Uložení zpět
    for i = 1:n
        app.bodies(i).pos = pos_new(i,:);
        app.bodies(i).vel = vel_new(i,:);

        app.bodies(i).trailx(end+1) = pos_new(i,1);
        app.bodies(i).traily(end+1) = pos_new(i,2);

        if numel(app.bodies(i).trailx) > app.trail_len
            app.bodies(i).trailx = app.bodies(i).trailx(end-app.trail_len+1:end);
            app.bodies(i).traily = app.bodies(i).traily(end-app.trail_len+1:end);
        end
    end

    app.time = app.time + app.dt;
end

% ==============================================================
% VÝPOČET ZRYCHLENÍ
% ==============================================================

function acc = compute_acc(pos, mass, Msun, G)
    n = size(pos,1);
    acc = zeros(n,2);
    eps2 = 1e-8;

    for i = 1:n
        % Gravitační vliv pevného Slunce v počátku
        r = -pos(i,:);
        d2 = r(1)^2 + r(2)^2 + eps2;
        d = sqrt(d2);
        acc(i,:) = acc(i,:) + G * Msun * r / (d^3);

        % Vzájemná gravitace těles
        for j = 1:n
            if j ~= i
                rij = pos(j,:) - pos(i,:);
                d2 = rij(1)^2 + rij(2)^2 + eps2;
                d = sqrt(d2);
                acc(i,:) = acc(i,:) + G * mass(j) * rij / (d^3);
            end
        end
    end
end

% ==============================================================
% PŘEKRESLENÍ
% ==============================================================

function app = redraw_scene(app)
    n = numel(app.bodies);

    for i = 1:n
        set(app.trailPlots(i), 'XData', app.bodies(i).trailx, 'YData', app.bodies(i).traily);
        set(app.bodyPlots(i),  'XData', app.bodies(i).pos(1), 'YData', app.bodies(i).pos(2));
        set(app.textPlots(i),  'Position', [app.bodies(i).pos(1)+0.18, app.bodies(i).pos(2)+0.18, 0]);
    end

    % Aktualizace Saturnova prstence
    th = linspace(0, 2*pi, 200);
    sat = app.bodies(4).pos;
    ring_x = sat(1) + 0.80*cos(th);
    ring_y = sat(2) + 0.28*sin(th);
    set(app.ringPlot, 'XData', ring_x, 'YData', ring_y);

    title(app.ax, sprintf('Planetární soustava | čas = %.2f roku', app.time), ...
        'Color', 'w', 'FontSize', 16);
end
