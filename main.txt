// ==UserScript==
// @name         AMNESIS
// @namespace    https://www.clutcher.io/
// @version      3.3.1
// @description  Right Shift GUI, configurable ESP, movement, aim, and world visuals.
// @icon         https://i.ibb.co/2YnYg7Ps/aasaf.png
// @match        https://www.clutcher.io/*
// @match        https://clutcher.io/*
// @match        file:///*/Clutcher-Offline.html
// @run-at       document-start
// @grant        none
// ==/UserScript==

(() => {
  'use strict';

  if (window.__amnesisGUI) return;
  window.__amnesisGUI = true;

  // Character rig property changed in the October game bundle.
  const characterAgent = entity => entity?.kx7Agent || entity?.cs2Agent;
  const characterEntities = g => {
    const entities = typeof g?.allEntities === 'function' ? [...g.allEntities()] :
      (Array.isArray(g?.botMgr?.bots) ? g.botMgr.bots : Object.values(g?.botMgr?.bots || {}));
    return entities.filter(entity => entity && entity !== g?.player &&
      (entity.team === 'T' || entity.team === 'CT'));
  };

  const KEY = 'clutcher.darkgui.heldskin.v1';
  const SKIN_KNIFE_MODELS = [
    'knife_bayonet','knife_bowie','knife_butterfly','knife_canis','knife_cord',
    'knife_css','knife_falchion','knife_flip','knife_gut','knife_karambit',
    'knife_kukri','knife_m9','knife_navaja','knife_nomad','knife_push',
    'knife_skeleton','knife_stiletto','knife_huntsman','knife_talon','knife_ursus'
  ];
  const SKIN_GLOVE_MODELS = [
    'handwrap','motorcycle','slick','specialist','sporty','bloodhound','brokenfang','hydra'
  ];

  const defaults = {
    x: null, y: null, tab: 'Visuals',
    compact: false,
    accent: '#ff3b4a',
    boneESP: false, boneESPTeam: false, arrowESP: false,
    espHealth: false, espName: false, espWeapon: false, espTracers: false,
    boxESP: false, cornerBoxESP: false, boxFill: false, boxFillOpacity: 12,
    headDotESP: false, headDotSize: 3, espDistance: false, espArmor: false,
    rainbowESP: false, rainbowSpeed: 60,
    espMaxDistance: 0, espOpacity: 100, espFontSize: 12, boxThickness: 2,
    espHealthColor: '#4dff6a', espArmorColor: '#6aa7d8', espHelmetColor: '#79c7ff',
    espHealthText: false, arrowDistance: false,
    radarESP: false, radarRange: 60, radarSize: 130,
    espEnemyColor: '#ff3b4a', espTeamColor: '#3d8bff', espTextColor: '#f5f5f8',
    espLineWidth: 2, espJointSize: 0,
    arrowVisibleColor: '#ff3548', arrowHiddenColor: '#e4e4ed',
    arrowRadiusPercent: 54, arrowSize: 15,
    silentAim: false, silentFov: 120, silentCircle: true,
    legitAimbot: false, legitAimMode: 'fire', legitAimBone: 'head',
    legitAimFov: 5, legitSmooth: 8, legitMaxSpeed: 180,
    legitAimMaxDistance: 150, legitCheckSmoke: true,
    legitDeadzone: false, legitDeadzoneFov: .3,
    legitTargetLock: false, legitHumanize: false, legitHumanizeAmount: .15,
    legitRcs: false, legitRcsPercent: 70,
    triggerDelay: false, triggerDelayMs: 80,
    legitAimCircle: false, legitDynamicSmooth: false,
    legitReactionDelay: false, legitReactionMs: 120,
    silentCircleColor: '#ff3b4a', silentCircleOpacity: 85, silentCircleWidth: 1.25,
    crosshairEnabled: false, crosshairColor: '#f5f5f8',
    crosshairSize: 8, crosshairGap: 4, crosshairWidth: 2,
    crosshairDot: false, crosshairDotSize: 2, crosshairOutline: false,
    customSkybox: false,
    customSkyboxUrl: 'https://cdn.sillyhack.com/assets/sky_oasis_new_night_fixed.jpg',
    screenVignette: false, vignetteColor: '#000000', vignetteOpacity: 45,
    worldSaturation: false, saturationPercent: 130,
    forceHitmarkers: false, localTracers: false, watermark: false,
    velocityHud: false, keybindList: false,
    fovOverride: false, cameraFov: 90,
    autoBhop: false, superBhop: false, superBhopSpeedPercent: 250,
    autoStrafe: false, strafeMode: 'wasd',
    groundSpeed: false, groundSpeedPercent: 150,
    crouchBoost: false, crouchSpeedPercent: 180,
    jumpBoost: false, jumpBoostPercent: 140,
    lowGravity: false, gravityPercent: 50, autoWalk: false,
    airBrake: false, airBrakeStrength: 85,
    edgeJump: false, momentumPreserve: false,
    ladderBoost: false, ladderSpeedPercent: 160,
    airAccelBoost: false, airAccelPercent: 160, autoAirCrouch: false,
    fallSpeedLimiter: false, maxFallSpeed: 8,
    reverseMovement: false, autoLandCrouch: false, landCrouchTime: 250,
    strafeCustomSpeed: false, strafeSpeedPercent: 100, rageSpeedPercent: 100, thirdPerson: false, thirdDistance: 3.30,
    fakeLag: false, fakeLagLimit: 120, fakeLagMode: 'always', fakeFreeze: false,
    antiAim: false,
    aaBase: 'target', aaYawMode: 'backward', aaYawOffset: 0,
    aaPitch: 'down', aaSpinSpeed: 90,
    aaJitter: 'off', aaJitterRange: 60, aaInterval: 120,
    aaDesync: 'off', aaDesyncAmount: 35, aaInvert: false,
    rageBot: false, rageResolver: true, rageFullMapWallbang: false,
    rageAutoCrouch: false, rageAutoStop: false, ragePriority: 'crosshair',
    rageForceHead: false, rageDamageBoost: false, rageDamagePercent: 150,
    rageTargetLock: false, rageBodyAimLowHp: false, rageBodyHp: 35,
    rageAutoJump: false,
    noSpread: false, noRecoil: false, infiniteAmmo: false, lethalShots: false,
    triggerBot: false, triggerRadius: 8,
    autoReload: false, autoReloadThreshold: 0,
    rageAutoFire: true, rageFov: 180, rageAreaHead: true, rageAreaNeckChest: true, rageAreaArms: false, rageAreaTorso: true, rageAreaLegs: false, rapidFire: false, hitsounds: false, hitsoundVolume: 100,
    vmChams: false, vmArms: true, vmWeapon: true, vmArmOpacity: 85, vmWeaponOpacity: 90, vmArmBrightness: 100,
    vmCustom: false, vmFov: 60, vmOffsetX: 0, vmOffsetY: 0, vmOffsetZ: 0,
    vmPitch: 0, vmYaw: 0, vmRoll: 0, vmBobPercent: 100, vmSwayPercent: 100,
    vmTint: '#ff3b4a', vmTintStrength: 35, worldEnabled: false,
    fullbright: false, fullbrightStrength: 170,
    removeFlash: false, hideViewmodel: false, noSmoke: false, noScopeOverlay: false,
    killCounter: false, damageCounter: false, hideNativeHud: false,
    noBlood: false, autoInspectKill: false,
    clockHud: false, fpsHud: false, coordinatesHud: false,
    skinChanger: false, skinCurrentMode: 'native',
    skinKnifeModel: 'native', skinGloveModel: 'native',
    skinRandomKnife: false, skinRandomGloves: false, skinBareHands: false,
    uiBlur: false, uiAnimations: true, uiTransparency: false, uiOpacity: 92,
    worldMode: 'day', worldNightStrength: 100,
    worldRain: false, worldRainIntensity: 55, worldRainSpeed: 65,
    worldRainColor: '#a9c4de', worldRainOpacity: 65,
    worldFog: false, worldFogMode: 'distance', worldFogNear: 18, worldFogFar: 125, worldFogDensity: 15, worldFogColor: '#b1c1d1'
  };
  const colorKeys = new Set([
    'accent', 'vmTint', 'worldFogColor', 'worldRainColor', 'espEnemyColor',
    'espTeamColor', 'espTextColor', 'espHealthColor', 'espArmorColor',
    'espHelmetColor', 'arrowVisibleColor', 'arrowHiddenColor',
    'silentCircleColor', 'crosshairColor', 'vignetteColor'
  ]);

  let saved = {};
  try {
    saved = JSON.parse(localStorage.getItem(KEY) || '{}');
  } catch {}

  const cfg = Object.fromEntries(Object.keys(defaults).map(key => {
    const fallback = defaults[key];
    const value = saved[key];
    const validPosition = (key === 'x' || key === 'y') &&
      (value === null || (typeof value === 'number' && Number.isFinite(value)));
    const validValue = value != null && typeof value === typeof fallback &&
      (typeof value !== 'number' || Number.isFinite(value)) &&
      (!colorKeys.has(key) || /^#[0-9a-f]{6}$/i.test(value));
    return [key, validPosition || validValue ? value : fallback];
  }));
  const tabs = ['Legit', 'Rage', 'Visuals', 'Movement', 'Misc', 'Skins', 'Settings'];

  // Module binds are stored separately so older exported configs remain compatible.
  const BIND_KEY = 'amnesis.module-keybinds.v1';
  const LEGACY_BIND_KEY = 'silly-v2.module-keybinds.v1';
  const bindable = new Set([
    'antiAim', 'fakeLag', 'rageBot', 'rapidFire', 'silentAim', 'legitAimbot',
    'legitRcs', 'triggerDelay', 'legitAimCircle', 'legitReactionDelay', 'noSpread',
    'autoBhop', 'superBhop', 'autoStrafe', 'vmChams', 'worldEnabled', 'boneESP', 'arrowESP',
    'thirdPerson', 'hitsounds', 'crosshairEnabled', 'boxESP', 'cornerBoxESP',
    'groundSpeed', 'crouchBoost', 'airBrake', 'triggerBot', 'autoReload',
    'velocityHud', 'keybindList', 'rainbowESP', 'fullbright', 'removeFlash',
    'hideViewmodel', 'vmCustom', 'noSmoke', 'noScopeOverlay', 'radarESP',
    'edgeJump', 'momentumPreserve', 'ladderBoost', 'airAccelBoost',
    'autoAirCrouch', 'fallSpeedLimiter', 'reverseMovement', 'autoLandCrouch',
    'hideNativeHud', 'noBlood', 'clockHud', 'fpsHud', 'coordinatesHud',
    'skinChanger', 'customSkybox', 'screenVignette', 'worldSaturation'
  ]);
  let binds = {};
  try {
    const stored = JSON.parse(localStorage.getItem(BIND_KEY) ||
      localStorage.getItem(LEGACY_BIND_KEY) || '{}');
    if (stored && typeof stored === 'object' && !Array.isArray(stored)) {
      for (const [key, code] of Object.entries(stored)) {
        if (bindable.has(key) && typeof code === 'string' && /^([A-Za-z0-9]+)$/.test(code) && code.length < 30 && code !== 'ShiftRight') binds[key] = code;
      }
    }
  } catch {}
  const saveBinds = () => { try { localStorage.setItem(BIND_KEY, JSON.stringify(binds)); } catch {} };
  let awaitingBind = null;
  const keyName = code => {
    if (!code) return '---';
    if (/^Key[A-Z]$/.test(code)) return code.slice(3);
    if (/^Digit[0-9]$/.test(code)) return code.slice(5);
    return ({Space:'SPACE',ShiftLeft:'L SHIFT',ControlLeft:'L CTRL',ControlRight:'R CTRL',
      AltLeft:'L ALT',AltRight:'R ALT',Mouse0:'MOUSE1',ArrowUp:'UP',ArrowDown:'DOWN',
      ArrowLeft:'LEFT',ArrowRight:'RIGHT'})[code] || code.replace(/^(Numpad)/,'NUM ' ).toUpperCase();
  };
  const refreshBindButtons = () => {
    if (!page) return;
    page.querySelectorAll('[data-bind-key]').forEach(button => {
      const key = button.dataset.bindKey;
      const listening = awaitingBind?.key === key;
      button.classList.toggle('listening', listening);
      button.textContent = listening ? 'PRESS KEY' : keyName(binds[key]);
      button.title = listening ? 'Press a key to bind, or ESC to clear / cancel' :
        ('Bind ' + key + '. ESC removes the current bind.');
    });
  };
  function bindButton(key, isHeader = false) {
    const button = document.createElement('button');
    button.type = 'button';
    button.className = 'amnesis-bind' + (isHeader ? ' bind-header' : '');
    button.dataset.bindKey = key;
    button.setAttribute('aria-label', 'Set keybind for ' + key);
    button.addEventListener('click', e => {
      e.preventDefault(); e.stopPropagation();
      awaitingBind = {key};
      refreshBindButtons();
      button.focus();
    });
    return button;
  }
  // Install before game/menu key listeners. Never toggle while menu is open.
  window.addEventListener('keydown', e => {
    if (!e.isTrusted) return;
    if (awaitingBind) {
      e.preventDefault(); e.stopImmediatePropagation();
      if (e.repeat) return;
      const key = awaitingBind.key;
      awaitingBind = null;
      if (e.code === 'Escape') delete binds[key];
      else if (e.code && e.code !== 'ShiftRight' && !/^(Meta|OS)/.test(e.code)) {
        // One key controls one module; rebinding removes an old assignment.
        for (const name of Object.keys(binds)) if (binds[name] === e.code) delete binds[name];
        binds[key] = e.code;
      }
      saveBinds(); refreshBindButtons();
      return;
    }
    if (e.repeat || uiOpen || e.code === 'ShiftRight' || e.code === 'Escape' ||
        e.ctrlKey || e.altKey || e.metaKey ||
        /^(INPUT|TEXTAREA|SELECT)$/.test(document.activeElement?.tagName || '') ||
        document.activeElement?.isContentEditable) return;
    const key = Object.keys(binds).find(name => binds[name] === e.code && bindable.has(name));
    if (!key) return;
    cfg[key] = !cfg[key];
    save();
    if (page) {
      page.querySelectorAll('input[data-setting-key]').forEach(input => {
        if (input.dataset.settingKey === key) input.checked = !!cfg[key];
      });
    }
    e.preventDefault();
    e.stopImmediatePropagation();
  }, true);


  if (!tabs.includes(cfg.tab)) cfg.tab = 'Visuals';

  const save = () => {
    try {
      localStorage.setItem(KEY, JSON.stringify(cfg));
    } catch {}
  };

  const configKeys = Object.keys(defaults).filter(key => !['x', 'y', 'tab', 'compact'].includes(key));

  const configEnums = {
    tab: tabs, strafeMode: ['wasd','legit','rage'], fakeLagMode: ['always','move','random'],
    legitAimMode: ['always','fire'], legitAimBone: ['head','chest','nearest'],
    ragePriority: ['crosshair','distance','health'],
    skinCurrentMode: ['native','default'],
    skinKnifeModel: ['native','default',...SKIN_KNIFE_MODELS],
    skinGloveModel: ['native','default',...SKIN_GLOVE_MODELS],
    aaBase: ['target','view'], aaYawMode: ['backward','forward','spin'],
    aaPitch: ['down','up','level','view'], aaJitter: ['off','center','offset','three','random'],
    aaDesync: ['off','static','alternate','sway'], worldMode: ['day','night'], worldFogMode: ['distance','exponential']
  };
  const configRanges = {
    silentFov:[20,500], legitAimFov:[0.5,30], legitSmooth:[1,30],
    legitMaxSpeed:[30,720], legitAimMaxDistance:[10,300],
    legitDeadzoneFov:[0,5], legitHumanizeAmount:[0,2], legitRcsPercent:[0,100],
    triggerDelayMs:[0,500], legitReactionMs:[0,1000],
    triggerRadius:[2,40], autoReloadThreshold:[0,15],
    superBhopSpeedPercent:[100,500],
    strafeSpeedPercent:[50,150], rageSpeedPercent:[50,150],
    thirdDistance:[.02,5], fakeLagLimit:[0,400], rageFov:[1,180],
    aaYawOffset:[-180,180], aaSpinSpeed:[-360,360], aaJitterRange:[0,180],
    aaInterval:[40,1000], aaDesyncAmount:[0,60], hitsoundVolume:[0,100],
    silentCircleOpacity:[10,100], silentCircleWidth:[0.5,5],
    crosshairSize:[2,30], crosshairGap:[0,20], crosshairWidth:[1,6],
    crosshairDotSize:[1,6], cameraFov:[60,130],
    vignetteOpacity:[0,100], saturationPercent:[0,300],
    boxFillOpacity:[1,50], headDotSize:[1,8], rainbowSpeed:[10,200],
    espMaxDistance:[0,300], espOpacity:[10,100], espFontSize:[8,24], boxThickness:[0.5,6],
    radarRange:[10,200], radarSize:[80,220],
    espLineWidth:[0.5,6], espJointSize:[0,6], arrowRadiusPercent:[25,90], arrowSize:[8,30],
    groundSpeedPercent:[100,300], crouchSpeedPercent:[100,300],
    jumpBoostPercent:[100,300], gravityPercent:[0,100], airBrakeStrength:[10,100],
    ladderSpeedPercent:[100,300], airAccelPercent:[100,300],
    maxFallSpeed:[2,30], landCrouchTime:[50,1000],
    vmArmOpacity:[0,100], vmWeaponOpacity:[0,100], vmTintStrength:[0,100], vmArmBrightness:[0,300],
    vmFov:[54,80], vmOffsetX:[-3,3], vmOffsetY:[-3,3], vmOffsetZ:[-3,3],
    vmPitch:[-30,30], vmYaw:[-30,30], vmRoll:[-45,45],
    vmBobPercent:[0,200], vmSwayPercent:[0,200],
    rageDamagePercent:[10,500], rageBodyHp:[1,100],
    uiOpacity:[40,100], fullbrightStrength:[100,250], worldNightStrength:[0,100], worldRainIntensity:[0,100], worldRainSpeed:[10,100], worldRainOpacity:[10,100],
    worldFogDensity:[0,100], worldFogNear:[0,250], worldFogFar:[10,500]
  };

  function parseConfig(text) {
    const data = JSON.parse(text.replace(/^\uFEFF/, ''));
    if (!data || typeof data !== 'object' || Array.isArray(data)) throw Error('Config must contain a settings object.');
    if (data.format !== undefined && !['amnesis','silly-v2'].includes(data.format)) throw Error('Unsupported config format.');
    if (data.version !== undefined && ['amnesis','silly-v2'].includes(data.format) && data.version !== 1) throw Error('Unsupported config version.');
    const settings = data.settings ?? data;
    if (!settings || typeof settings !== 'object' || Array.isArray(settings)) throw Error('Missing settings object.');
    const result = {};
    for (const key of configKeys) {
      if (!Object.prototype.hasOwnProperty.call(settings, key)) continue;
      const value = settings[key];
      if (key === 'x' || key === 'y') {
        if (value !== null && (typeof value !== 'number' || !Number.isFinite(value))) throw Error('Invalid ' + key);
      } else if (typeof value !== typeof defaults[key]) throw Error('Invalid type for ' + key);
      if (typeof value === 'number' && !Number.isFinite(value)) throw Error('Invalid number for ' + key);
      if (configEnums[key] && !configEnums[key].includes(value)) throw Error('Invalid option for ' + key);
      if (colorKeys.has(key) &&
          !/^#[0-9a-f]{6}$/i.test(value)) throw Error('Invalid color for ' + key);
      if (configRanges[key] && (value < configRanges[key][0] || value > configRanges[key][1])) throw Error('Out-of-range value for ' + key);
      result[key] = value;
    }
    if (!Object.keys(result).length) throw Error('No recognized settings found.');
    return result;
  }

  function exportConfig() {
    const settings = Object.fromEntries(configKeys.map(key => [key, cfg[key]]));
    const blob = new Blob([JSON.stringify({format:'amnesis', version:1, settings}, null, 2)], {type:'application/json'});
    const url = URL.createObjectURL(blob);
    const link = document.createElement('a');
    link.href = url;
    link.download = 'AMNESIS.cfg';
    document.body.append(link);
    link.click();
    link.remove();
    setTimeout(() => URL.revokeObjectURL(url), 1000);
  }

  function configControls() {
    const panel = section('Configs', 'Import or export module settings and colors as a .cfg file. Imports apply immediately. Missing settings keep their current values.');
    const message = elem('p', 'note');
    message.setAttribute('role', 'status');
    const exportButton = elem('button', 'action', 'Export .cfg');
    exportButton.onclick = exportConfig;
    const importButton = elem('button', 'action', 'Import .cfg');
    const picker = elem('input');
    picker.type = 'file';
    picker.accept = '.cfg,application/json';
    picker.hidden = true;
    importButton.onclick = () => picker.click();
    picker.onchange = async () => {
      const file = picker.files?.[0];
      if (!file) return;
      try {
        if (file.size > 128 * 1024) throw Error('Config exceeds 128 KB.');
        const settings = parseConfig(await file.text());
        Object.assign(cfg, settings);
        save();
        box.style.setProperty('--accent', cfg.accent);
        renderNav();
        renderPage();
        const notice = elem('p', 'note', 'Imported ' + file.name + ' (' + Object.keys(settings).length + ' settings).');
        notice.setAttribute('role', 'status');
        page.prepend(notice);
      } catch (error) {
        message.textContent = 'Import failed: ' + error.message;
      } finally { picker.value = ''; }
    };
    panel.append(exportButton, importButton, picker, message);
  }

  // Artemis-compatible player_state transport. Local physics stays untouched.
  const fakeNet = new WeakMap();
  const fakeSnapshotKeys = ['x', 'y', 'z', 'yaw', 'pitch', 'crouch', 'crouching', 'onGround'];
  const snapshotState = state => Object.fromEntries(
    fakeSnapshotKeys.filter(key => key in state).map(key => [key, state[key]])
  );
  const nativeSocketSend = window.WebSocket?.prototype.send;
  if (nativeSocketSend) {
    window.WebSocket.prototype.send = function(data) {
      if (typeof data !== 'string') return nativeSocketSend.call(this, data);
      let packet;
      try { packet = JSON.parse(data); } catch { return nativeSocketSend.call(this, data); }
      if (packet?.type !== 'player_state' || !packet.state ||
          !['x', 'y', 'z'].every(key => Number.isFinite(packet.state[key]))) {
        return nativeSocketSend.call(this, data);
      }
      let net = fakeNet.get(this);
      if (!net) {
        net = { freeze: null, lag: null, last: null, at: 0, interval: 0, mode: '' };
        fakeNet.set(this, net);
      }
      const state = packet.state;
      const now = performance.now();
      const rawLimit = Number(cfg.fakeLagLimit);
      const limit = Number.isFinite(rawLimit) ? Math.max(0, Math.min(400, rawLimit)) : 120;
      const mode = ['always', 'move', 'random'].includes(cfg.fakeLagMode) ? cfg.fakeLagMode : 'always';
      const active = state.alive !== false && window.game?.player?.alive !== false;
      const moved = net.last && Math.hypot(state.x-net.last.x, state.z-net.last.z) > 0.015;
      const moving = Math.hypot(Number(state.vx)||0, Number(state.vz)||0) > 0.5 || moved;
      const teleported = net.last && Math.hypot(state.x-net.last.x, state.y-net.last.y, state.z-net.last.z) > 5;
      net.last = { x: state.x, y: state.y, z: state.z };
      if (!active || teleported) { net.freeze = null; net.lag = null; }
      if (!cfg.fakeFreeze || !active) net.freeze = null;
      if (!cfg.fakeLag || !active || (mode === 'move' && !moving)) net.lag = null;
      let snap = null;
      if (cfg.fakeFreeze && active) {
        if (!net.freeze) net.freeze = snapshotState(state);
        snap = net.freeze;
        net.lag = null;
      } else if (cfg.fakeLag && active && limit > 0 && (mode !== 'move' || moving)) {
        if (!net.lag || now-net.at >= net.interval || net.mode !== mode || net.limit !== limit) {
          net.lag = snapshotState(state);
          net.at = now;
          net.interval = mode === 'random' ? Math.max(0, limit+(Math.random()*2-1)*Math.min(80,limit*0.25)) : limit;
          net.limit = limit;
          net.mode = mode;
        }
        snap = net.lag;
      }
      if (!snap) return nativeSocketSend.call(this, data);
      const reported = { ...state, ...snap, vx: 0, vy: 0, vz: 0, walking: false };
      // Preserve any live anti-aim angles already applied by the multiplayer client.
      if (cfg.antiAim) { reported.yaw = state.yaw; reported.pitch = state.pitch; }
      return nativeSocketSend.call(this, JSON.stringify({ ...packet, state: reported }));
    };
  }

  const norm = s => String(s || '').toLowerCase().trim().replace(/^weapon_/, '');
  const titlecase = s => String(s || '').replace(/[_-]/g, ' ').replace(/\b\w/g, c => c.toUpperCase());

  const knifeRE = /knife|bayonet|karambit|dagger|falchion|bowie|huntsman|navaja|nomad|paracord|skeleton|stiletto|survival|talon|ursus|kukri/i;
  const isKnife = s => knifeRE.test(norm(s));
  let uiOpen = false;
  let root, box, nav, page, pageBody;
  let lockEl, hadLock = false, drag = null;

  // =====================================================
  // HITSOUNDS: custom local audio on confirmed kills
  // =====================================================
  let hitsoundURL = null;
  let hitsoundName = '';
  const hitsoundPlayers = new Set();
  const sessionStats = {kills:0, damage:0};
  let pendingInspect = null;

  function playHitsound() {
    if (!cfg.hitsounds || !hitsoundURL) return;
    const audio = new Audio(hitsoundURL);
    audio.volume = Math.max(0, Math.min(1, Number(cfg.hitsoundVolume) / 100));
    hitsoundPlayers.add(audio);
    const cleanup = () => hitsoundPlayers.delete(audio);
    audio.addEventListener('ended', cleanup, {once:true});
    audio.addEventListener('error', cleanup, {once:true});
    audio.play().catch(cleanup);
  }

  function hitsoundDatabase() {
    return new Promise((resolve, reject) => {
      const request = indexedDB.open('SillyV2Hitsounds', 1);
      request.onupgradeneeded = () => request.result.createObjectStore('audio');
      request.onsuccess = () => resolve(request.result);
      request.onerror = () => reject(request.error);
    });
  }

  async function storeHitsound(file) {
    const db = await hitsoundDatabase();
    try {
      await new Promise((resolve, reject) => {
        const tx = db.transaction('audio', 'readwrite');
        tx.objectStore('audio').put({blob:file, name:file.name}, 'selected');
        tx.oncomplete = resolve;
        tx.onerror = () => reject(tx.error);
        tx.onabort = () => reject(tx.error);
      });
    } finally { db.close(); }
  }

  async function loadHitsound() {
    try {
      const db = await hitsoundDatabase();
      let item;
      try {
        item = await new Promise((resolve, reject) => {
          const tx = db.transaction('audio', 'readonly');
          const req = tx.objectStore('audio').get('selected');
          req.onsuccess = () => resolve(req.result);
          req.onerror = () => reject(req.error);
        });
      } finally { db.close(); }
      if (item?.blob) setHitsound(item.blob, item.name);
    } catch (error) { console.warn('[Hitsounds] Could not restore audio:', error); }
  }

  function setHitsound(blob, name) {
    const previous = hitsoundURL;
    hitsoundURL = URL.createObjectURL(blob);
    hitsoundName = name || 'Custom audio';
    if (previous) URL.revokeObjectURL(previous);
  }

  function hitsoundControls() {
    const panel = section('Hitsounds', 'Play your own audio file whenever you kill an enemy.');
    check(panel, 'Enable Hitsounds', 'hitsounds');
    aaRange(panel, 'Volume', 'hitsoundVolume', 0, 100, 1, '%');
    const fileRow = elem('div', 'field');
    const filename = elem('span', '', hitsoundName || 'No audio selected');
    const picker = elem('input');
    picker.type = 'file';
    picker.accept = 'audio/*,.mp3,.wav,.ogg,.flac,.m4a,.aac,.opus';
    picker.hidden = true;
    const importButton = elem('button', 'action', 'Import audio');
    const previewButton = elem('button', 'action', 'Preview');
    const notice = elem('p', 'note');
    importButton.onclick = () => picker.click();
    picker.onchange = async () => {
      const file = picker.files?.[0];
      if (!file) return;
      try {
        if (file.size > 25 * 1024 * 1024) throw Error('Audio must be under 25 MB.');
        if (!file.type.startsWith('audio/') && !/\.(mp3|wav|ogg|flac|m4a|aac|opus)$/i.test(file.name))
          throw Error('Choose an audio file.');
        setHitsound(file, file.name);
        filename.textContent = hitsoundName;
        await storeHitsound(file);
        notice.textContent = 'Saved ' + file.name + ' for future visits.';
      } catch (error) { notice.textContent = 'Import: ' + error.message; }
      finally { picker.value = ''; }
    };
    previewButton.onclick = () => {
      if (!hitsoundURL) { notice.textContent = 'Import an audio file first.'; return; }
      const audio = new Audio(hitsoundURL);
      audio.volume = Math.max(0, Math.min(1, Number(cfg.hitsoundVolume) / 100));
      audio.play().catch(error => { notice.textContent = 'Preview failed: ' + error.message; });
    };
    fileRow.append(filename, importButton, previewButton, picker);
    panel.append(fileRow, notice);
  }

  loadHitsound().then(() => {
    if (cfg.tab === 'Misc' && page) renderPage();
  });

  // =====================================================
  // LOCAL VIEWMODEL CHAMS + WORLD VISUAL SETTINGS
  // =====================================================
  // Only traverse the local first-person viewmodel scene. Never alter
  // game.scene materials, enemy entities, or shared source materials.
  const vmOriginals = new Map();
  let vmTrackedScene = null;
  const worldOriginals = new WeakMap();
  const worldLightOriginals = new Map();
  const worldRendererOriginals = new Map();
  let worldActiveScene = null;
  const clamp01 = value => Math.max(0, Math.min(1, Number(value) || 0));

  // Match actual viewmodel parts, rather than broad ancestor names like
  // "skin" (which also appears in weapon/skin hierarchies).
  const vmArmName = /(?:^|[^a-z])(?:arms?|hands?|gloves?|sleeves?|wrists?|fingers?|forearms?)(?:$|[^a-z])/i;
  const vmWeaponName = /(?:^|[^a-z])(?:weapon|gun|rifle|pistol|sniper|shotgun|smg|knife|blade|magazine|receiver|barrel|stock|scope|sight|slide|trigger|grip|silencer|suppressor|muzzle|bullet|shell|bolt|optic|ak47|m4a1|m4|awp|glock|usp|deagle|famas|galil|aug|sg553|mp5|mp7|mp9|ump|p90|nova|xm1014|m249|negev)(?:$|[^a-z])/i;

  function classifyViewMesh(mesh) {
    // A weapon's own mesh/material name wins over a generic named parent.
    // Without confident evidence of arms, leave it categorized as weapon:
    // when Weapon Chams is OFF it must never receive arm opacity.
    const own = [mesh.name, mesh.material?.name, mesh.userData?.name]
      .filter(v => typeof v === 'string').join(' ');
    if (vmWeaponName.test(own)) return 'weapon';
    if (vmArmName.test(own)) return 'arms';
    let node = mesh.parent;
    for (let depth = 0; node && depth < 7; node = node.parent, depth++) {
      const name = String(node.name || '');
      if (vmWeaponName.test(name)) return 'weapon';
      if (vmArmName.test(name)) return 'arms';
    }
    // Some viewmodel meshes use unnamed geometries, but arm skinned meshes
    // commonly expose an armature. Do not infer arms from generic "skin".
    const boneNames = mesh.skeleton?.bones?.map(b => b.name).join(' ') || '';
    if (vmWeaponName.test(boneNames)) return 'weapon';
    if (vmArmName.test(boneNames)) return 'arms';
    return 'weapon';
  }
  function restoreViewmodel() {
    for (const [mesh, entry] of vmOriginals) {
      if (mesh.material === entry.cloned) mesh.material = entry.original;
      for (const mat of entry.owned) mat.dispose?.();
    }
    vmOriginals.clear();
    vmTrackedScene = null;
  }
  function updateViewmodelChams(g) {
    const scene = g?.viewmodel?.scene;
    if (!cfg.vmChams || !scene) { if (vmOriginals.size) restoreViewmodel(); return; }
    if (vmTrackedScene && vmTrackedScene !== scene) restoreViewmodel();
    vmTrackedScene = scene;
    const present = new Set();
    scene.traverse?.(mesh => {
      if (!mesh?.isMesh || !mesh.material) return;
      present.add(mesh);
      let entry = vmOriginals.get(mesh);
      if (entry && mesh.material !== entry.cloned) {
        for (const mat of entry.owned) mat.dispose?.();
        vmOriginals.delete(mesh);
        entry = null;
      }
      const arms = classifyViewMesh(mesh) === 'arms';
      const enabled = arms ? cfg.vmArms : cfg.vmWeapon;
      if (!enabled) {
        if (entry) {
          mesh.material = entry.original;
          entry.owned.forEach(m => m.dispose?.());
          vmOriginals.delete(mesh);
        }
        return;
      }
      if (!entry) {
        const original = mesh.material;
        const owned = (Array.isArray(original) ? original : [original]).map(m => m?.clone?.() || m);
        const cloned = Array.isArray(original) ? owned : owned[0];
        entry = {original, owned, cloned};
        vmOriginals.set(mesh, entry);
        mesh.material = cloned;
      }
      const opacity = clamp01((arms ? cfg.vmArmOpacity : cfg.vmWeaponOpacity) / 100);
      for (const mat of entry.owned) {
        if (!mat) continue;
        mat.opacity = opacity;
        mat.transparent = opacity < 0.999 || mat.transparent;
        mat.depthWrite = opacity >= 0.999;
        // Keep all client-selected maps, skins, and textures. Blend only
        // the cloned material tint to preserve the user's appearance.
        const source = (Array.isArray(entry.original) ? entry.original : [entry.original])
          [entry.owned.indexOf(mat)];
        if (mat.color && source?.color) {
          mat.color.copy(source.color);
          if (cfg.vmTintStrength > 0) {
            mat.color.lerp(new mat.color.constructor(cfg.vmTint), clamp01(cfg.vmTintStrength / 100));
          }
        }
        // Arms-only brightness: 100% is original; up to 300% adds
        // stronger emissive illumination for very dark night scenes.
        // Restore the source material values each frame (no cumulative glow).
        // Preserve skin textures by using the original diffuse map as emissive.
        if (arms) {
          const brightness = Math.max(0, Math.min(300, Number(cfg.vmArmBrightness) || 0)) / 100;
          if (brightness < 1 && mat.color) mat.color.multiplyScalar(brightness);
          if (mat.emissive && source?.emissive) {
            mat.emissive.copy(source.emissive);
            mat.emissiveIntensity = source.emissiveIntensity ?? 1;
            mat.emissiveMap = source.emissiveMap || null;
            if (brightness > 1) {
              const lift = brightness - 1;
              // Mimic the base color's hue without adding a fullscreen filter.
              const glow = mat.color || source.color;
              if (glow) mat.emissive.add(glow.clone().multiplyScalar(lift * 1.8));
              mat.emissiveMap = source.emissiveMap || source.map || null;
              mat.emissiveIntensity = Math.max(1, 1 + lift * 1.5);
            }
          }
        }
        mat.needsUpdate = true;
      }
    });
    for (const [mesh, entry] of vmOriginals) {
      if (!present.has(mesh)) {
        if (mesh.material === entry.cloned) mesh.material = entry.original;
        entry.owned.forEach(m => m.dispose?.());
        vmOriginals.delete(mesh);
      }
    }
  }

  const customViewmodelHooks = new WeakSet();
  const customViewmodelStates = new WeakMap();

  function applyCustomViewmodel(g, vm) {
    let state = customViewmodelStates.get(vm);
    if (!state) {
      state = {active:false,cameraFov:vm.camera?.fov,key:''};
      customViewmodelStates.set(vm,state);
    }
    const kx = g.kx7;
    if (!cfg.vmCustom) {
      if (state.active) {
        g.hud?.applyViewmodel?.();
        kx?.root?.rotation?.set?.(0,0,0);
        if (vm.camera && Number.isFinite(state.cameraFov)) {
          vm.camera.fov=state.cameraFov;
          vm.camera.updateProjectionMatrix?.();
        }
        state.active=false;
        state.key='';
      }
      return;
    }
    if (!state.active) state.cameraFov=vm.camera?.fov;
    state.active=true;
    const native=g.hud?.vm||{};
    const fov=Math.max(54,Math.min(80,Number(cfg.vmFov)||60));
    const ox=Math.max(-3,Math.min(3,Number(cfg.vmOffsetX)||0));
    const oy=Math.max(-3,Math.min(3,Number(cfg.vmOffsetY)||0));
    const oz=Math.max(-3,Math.min(3,Number(cfg.vmOffsetZ)||0));
    const bob=Math.max(0,Math.min(200,Number(cfg.vmBobPercent)||0))/100;
    const sway=Math.max(0,Math.min(200,Number(cfg.vmSwayPercent)||0))/100;
    const pitch=Math.max(-30,Math.min(30,Number(cfg.vmPitch)||0))*Math.PI/180;
    const yaw=Math.max(-30,Math.min(30,Number(cfg.vmYaw)||0))*Math.PI/180;
    const roll=Math.max(-45,Math.min(45,Number(cfg.vmRoll)||0))*Math.PI/180;
    const kxVisible=!!(kx?.root && kx.root.visible!==false);

    if (kxVisible) {
      const px=(Number(native.x)||0)+ox;
      const py=(Number(native.y)||0)+oy;
      const pz=(Number(native.z)||0)+oz;
      const bobBase=native.bob===0?0:(Number(native.bobAmt??60)/100);
      const swayBase=native.sway===0?0:(Number(native.swayAmt??45)/100);
      const key=[fov,px,py,pz,bobBase*bob,swayBase*sway].join('|');
      if (state.key!==key) {
        kx.setPlacement?.(fov,px,py,pz);
        kx.setMotion?.(bobBase*bob,swayBase*sway);
        state.key=key;
      }
      kx.root.rotation?.set?.(pitch,yaw,roll);
      const converted=2*Math.atan(Math.tan(fov*Math.PI/360)*.75)*180/Math.PI;
      if (vm.camera && Math.abs(vm.camera.fov-converted)>.001) {
        vm.camera.fov=converted;
        vm.camera.updateProjectionMatrix?.();
      }
    } else {
      const root=vm.root;
      if (root?.position) {
        root.position.x+=ox*.0254;
        root.position.y+=oz*.0254;
        root.position.z-=oy*.0254;
        const bobX=Math.cos(Number(vm.bobT)||0)*.009*(Number(vm.bobAmt)||0);
        const bobY=Math.sin((Number(vm.bobT)||0)*2)*.012*(Number(vm.bobAmt)||0);
        root.position.x+=bobX*(bob-1)+(Number(vm.swayX)||0)*(sway-1);
        root.position.y+=bobY*(bob-1)+(Number(vm.swayY)||0)*(sway-1);
      }
      if (root?.rotation) {
        root.rotation.x+=pitch; root.rotation.y+=yaw; root.rotation.z+=roll;
      }
      if (vm.camera && Math.abs(vm.camera.fov-fov)>.001) {
        vm.camera.fov=fov;
        vm.camera.updateProjectionMatrix?.();
      }
    }
  }

  function ensureCustomViewmodelHook() {
    const g=window.game, vm=g?.viewmodel;
    if (!vm || typeof vm.update!=='function' || customViewmodelHooks.has(vm)) return;
    const original=vm.update;
    vm.update=function(...args) {
      const result=original.apply(this,args);
      applyCustomViewmodel(g,this);
      return result;
    };
    customViewmodelHooks.add(vm);
  }

  setInterval(ensureCustomViewmodelHook,100);

  // Day/Night is binary. Never draw an HTML overlay over the player or HUD.
  const worldNightAmount = () => cfg.worldMode === 'night' ? 1 : 0;

  const skyMaterialOriginals = new Map();
  let worldBackgroundOriginal = null;
  function restoreSkyMaterials() {
    for (const [mat, data] of skyMaterialOriginals) {
      if (data.color && mat.color) mat.color.copy(data.color);
      if (data.opacity !== undefined) mat.opacity = data.opacity;
      mat.needsUpdate = true;
    }
    skyMaterialOriginals.clear();
  }
  // Oasis and Dune share the proven Oasis CDN night skybox while
  // keeping the game's native projection/UV mapping.
  const OASIS_NIGHT_SKY = 'https://cdn.sillyhack.com/assets/sky_oasis_new_night_fixed.jpg';
  const oasisSkyOriginals = new Map();
  const duneSkyOriginals = new Map();
  let oasisNightImage = null;
  let oasisSkyLoadStarted = false;
  let oasisSkyFailed = false;
  let oasisSkyLastScan = 0;
  const oasisSource = /sky_oasis_new_(?:low|env_mobile)\.jpg/i;
  const duneSource = /sky_(?:dune_new|dust2?|dust_2)_(?:low|env_mobile)\.jpg/i;
  const customSkyOriginals = new Map();
  let customSkyImage=null,customSkyUrl='',customSkyLoading=false;

  function restoreCustomSkybox() {
    for (const [texture,saved] of customSkyOriginals) {
      const original=saved.image||saved.data;
      if (original&&texture.image!==original) {
        texture.image=original;texture.needsUpdate=true;
      }
    }
    customSkyOriginals.clear();
  }

  function updateCustomSkyboxImage() {
    const url=String(cfg.customSkyboxUrl||'').trim();
    if (!cfg.customSkybox||!url) {
      if (customSkyOriginals.size) restoreCustomSkybox();
      customSkyImage=null;customSkyUrl='';customSkyLoading=false;return;
    }
    if (url===customSkyUrl&&(customSkyImage||customSkyLoading)) return;
    restoreCustomSkybox();customSkyImage=null;customSkyUrl=url;customSkyLoading=true;
    const image=new Image();image.crossOrigin='anonymous';
    image.onload=()=>{
      if (customSkyUrl===url) customSkyImage=image;
      customSkyLoading=false;
    };
    image.onerror=()=>{if(customSkyUrl===url)customSkyLoading=false;};
    image.src=url;
  }

  function applyCustomSkybox(texture) {
    if (!cfg.customSkybox||!customSkyImage||!texture?.isTexture||texture.isCubeTexture) return;
    if (!customSkyOriginals.has(texture)) {
      const source=oasisTextureSource(texture);
      if (!/sky|environment|cubemap/i.test(source)&&
          !oasisSkyOriginals.has(texture)&&!duneSkyOriginals.has(texture)) return;
      customSkyOriginals.set(texture,{image:texture.image,data:texture.source?.data});
    }
    if (texture.image!==customSkyImage) {
      texture.image=customSkyImage;texture.needsUpdate=true;
    }
  }

  function applyDuneTexture(texture, night) {
    if (!texture?.isTexture) return;
    let saved = duneSkyOriginals.get(texture);
    if (!saved) {
      if (!duneSource.test(oasisTextureSource(texture))) return;
      saved = {image:texture.image, data:texture.source?.data};
      duneSkyOriginals.set(texture, saved);
    }
    const next = night && oasisNightImage ? oasisNightImage : (saved.image || saved.data);
    if (!next || texture.image === next) return;
    texture.image = next;
    texture.needsUpdate = true;
  }

  function restoreDuneSky() {
    for (const [texture, saved] of duneSkyOriginals) {
      const original = saved.image || saved.data;
      if (original && texture.image !== original) {
        texture.image = original;
        texture.needsUpdate = true;
      }
    }
    duneSkyOriginals.clear();
  }


  function loadOasisNightSky() {
    if (oasisNightImage || oasisSkyLoadStarted || oasisSkyFailed) return;
    oasisSkyLoadStarted = true;
    const image = new Image();
    image.crossOrigin = 'anonymous';
    image.onload = () => {
      oasisNightImage = image;
      console.info('[AMNESIS World] Oasis nighttime skybox loaded');
    };
    image.onerror = () => {
      oasisSkyFailed = true;
      console.warn('[AMNESIS World] Night skybox failed to load; check CDN URL and CORS:', OASIS_NIGHT_SKY);
    };
    image.src = OASIS_NIGHT_SKY;
  }

  function oasisTextureSource(texture) {
    const image = texture?.image || texture?.source?.data;
    return String(image?.currentSrc || image?.src || texture?.userData?.url || '');
  }

  function applyOasisTexture(texture, night) {
    if (!texture?.isTexture) return;
    let saved = oasisSkyOriginals.get(texture);
    if (!saved) {
      if (!oasisSource.test(oasisTextureSource(texture))) return;
      saved = {image:texture.image, data:texture.source?.data};
      oasisSkyOriginals.set(texture, saved);
    }
    const next = night && oasisNightImage ? oasisNightImage : (saved.image || saved.data);
    if (!next || texture.image === next) return;
    texture.image = next;
    texture.needsUpdate = true;
  }

  function restoreOasisSky() {
    for (const [texture, saved] of oasisSkyOriginals) {
      if (saved.image || saved.data) {
        texture.image = saved.image || saved.data;
        texture.needsUpdate = true;
      }
    }
    oasisSkyOriginals.clear();
  }

  function updateOasisSky(g, night) {
    updateCustomSkyboxImage();
    if (night) loadOasisNightSky();
    // This avoids unnecessarily walking the entire scene every rendered frame.
    const now = performance.now();
    if (now - oasisSkyLastScan < 500) return;
    oasisSkyLastScan = now;
    const seen = new Set();
    const visitTexture = texture => {
      if (!texture || seen.has(texture)) return;
      seen.add(texture);
      applyOasisTexture(texture, night);
      applyDuneTexture(texture, night);
      applyCustomSkybox(texture);
    };
    const visitMaterial = material => {
      if (!material) return;
      for (const mat of (Array.isArray(material) ? material : [material])) {
        if (!mat) continue;
        for (const key of ['map','envMap','backgroundMap','emissiveMap','alphaMap']) visitTexture(mat[key]);
        for (const uniform of Object.values(mat.uniforms || {})) {
          if (uniform?.value?.isTexture) visitTexture(uniform.value);
        }
      }
    };
    // Some maps put their sky in a separate scene; never touch other textures.
    for (const scene of [g.scene,g.skyScene,g._skyScene,g.backgroundScene,g.sky?.scene]) {
      if (!scene) continue;
      visitTexture(scene.background);
      scene.traverse?.(node => visitMaterial(node.material));
    }
    visitMaterial(g.sky?.material);
    visitTexture(g.sky?.texture);
    visitTexture(g.skybox?.texture);
  }

  function restoreWorld() {
    restoreCustomSkybox();
    restoreOasisSky();
    restoreDuneSky();
    restoreSkyMaterials();
    if (worldActiveScene) {
      const original = worldOriginals.get(worldActiveScene);
      if (original) {
        worldActiveScene.fog = original.fog;
        if (original.backgroundColor && worldActiveScene.background?.isColor)
          worldActiveScene.background.copy(original.backgroundColor);
        if (original.backgroundIntensity !== undefined)
          worldActiveScene.backgroundIntensity = original.backgroundIntensity;
      }
    }
    for (const [light, state] of worldLightOriginals) {
      light.intensity = state.intensity;
      if (state.color && light.color) light.color.copy(state.color);
    }
    worldLightOriginals.clear();
    for (const [renderer, state] of worldRendererOriginals) {
      if (state.toneMappingExposure !== undefined && Number.isFinite(state.toneMappingExposure))
        renderer.toneMappingExposure = state.toneMappingExposure;
    }
    worldRendererOriginals.clear();
    worldActiveScene = null;
  }
  function updateWorldVisuals(g) {
    const scene = g?.scene;
    const renderer = g?.renderer;
    const worldEnabled = !!cfg.worldEnabled;
    if ((!worldEnabled && !cfg.fullbright && !cfg.customSkybox) || !scene) {
      if (worldActiveScene) restoreWorld();
      return;
    }
    if (worldActiveScene && worldActiveScene !== scene) restoreWorld();
    worldActiveScene = scene;
    if (!worldOriginals.has(scene)) worldOriginals.set(scene, {
      fog:scene.fog,
      backgroundColor:scene.background?.isColor ? scene.background.clone() : null,
      backgroundIntensity:scene.backgroundIntensity
    });
    if (renderer && !worldRendererOriginals.has(renderer)) {
      worldRendererOriginals.set(renderer, {
        toneMappingExposure:Number.isFinite(renderer.toneMappingExposure) ? renderer.toneMappingExposure : undefined
      });
    }
    const original = worldOriginals.get(scene);
    const nightEnabled = worldEnabled && worldNightAmount() === 1;
    const night = nightEnabled
      ? clamp01(Number(cfg.worldNightStrength) / 100)
      : 0;
    updateOasisSky(g, nightEnabled);
    // Both maps use the same original stronger night exposure and light curve.

    // Darken actual scene output at night instead of drawing an HTML overlay.
    if (renderer && Number.isFinite(worldRendererOriginals.get(renderer)?.toneMappingExposure)) {
      const base = worldRendererOriginals.get(renderer).toneMappingExposure;
      renderer.toneMappingExposure = Math.max(0.35, base * (1 - 0.62 * night));
      if (cfg.fullbright) {
        const strength = Math.max(100, Math.min(250,
          Number(cfg.fullbrightStrength) || 170)) / 100;
        renderer.toneMappingExposure = Math.max(renderer.toneMappingExposure,
          base * strength);
      }
    }

    // Prefer native sky/background changes. If the background is a Color,
    // strongly darken it and bias it cooler at night.
    if (scene.background?.isColor && original.backgroundColor) {
      scene.background.copy(original.backgroundColor);
      scene.background.r *= Math.max(0.01, 1 - 0.985 * night);
      scene.background.g *= Math.max(0.02, 1 - 0.955 * night);
      scene.background.b *= Math.max(0.08, 1 - 0.78 * night);
    }
    if ('backgroundIntensity' in scene && Number.isFinite(original.backgroundIntensity)) {
      scene.backgroundIntensity = Math.max(0.04, original.backgroundIntensity * (1 - 0.94 * night));
    }

    scene.traverse?.(node => {
      if (!node?.isMesh) return;
      const name = String(node.name || '').toLowerCase();
      if (!/(?:^|[_ .-])(sky|skydome|skybox|clouds?|atmosphere|sun|stars?)(?:$|[_ .-])/.test(name)) return;
      for (const mat of (Array.isArray(node.material) ? node.material : [node.material])) {
        // Do not double-darken the custom nighttime sky textures.
        if (night && [mat?.map, mat?.envMap].some(t => t && (oasisSkyOriginals.has(t) || duneSkyOriginals.has(t)))) continue;
        if (!mat?.color) continue;
        if (!skyMaterialOriginals.has(mat)) skyMaterialOriginals.set(mat, {color:mat.color.clone()});
        const base = skyMaterialOriginals.get(mat).color;
        mat.color.copy(base);
        mat.color.r *= Math.max(0.01, 1 - 0.985 * night);
        mat.color.g *= Math.max(0.02, 1 - 0.955 * night);
        mat.color.b *= Math.max(0.08, 1 - 0.78 * night);

        mat.needsUpdate = true;
      }
    });

    // Lighting affects level surfaces and makes midnight actually look dark.
    scene.traverse?.(node => {
      if (!node?.isLight || !Number.isFinite(node.intensity)) return;
      if (!worldLightOriginals.has(node))
        worldLightOriginals.set(node, {intensity:node.intensity, color:node.color?.clone?.()});
      const state = worldLightOriginals.get(node);
      node.intensity = Math.max(0, state.intensity * (1 - 0.9 * night));
      if (cfg.fullbright) {
        const strength = Math.max(100, Math.min(250,
          Number(cfg.fullbrightStrength) || 170)) / 100;
        node.intensity = Math.max(node.intensity, state.intensity * strength);
      }
      if (state.color && node.color) {
        node.color.copy(state.color);
        node.color.r *= Math.max(0.25, 1 - 0.55 * night);
        node.color.g *= Math.max(0.35, 1 - 0.42 * night);
        node.color.b *= Math.max(0.60, 1 - 0.12 * night);
      }
    });

    // If there is no usable sky object, the stronger exposure and lighting
    // curve still provide a visible night-time scene.

    // Three.js native fog uniforms/shader path. Does not need window.THREE:
    // Fog and FogExp2 are property-based types in the renderer.
    // This is real depth-based scene fog, NOT a canvas/HTML overlay.
    if (worldEnabled && cfg.worldFog) {
      if (!scene.fog?.__amnesisFog || scene.fog.__amnesisMode !== cfg.worldFogMode) {
        let color = null;
        if (original.fog?.color?.clone) color = original.fog.color.clone();
        else if (scene.background?.isColor) color = scene.background.clone();
        else {
          scene.traverse?.(node => {
            if (!color && node?.isLight && node.color?.clone) color = node.color.clone();
          });
        }
        if (color && typeof color.set === 'function') {
          color.set(cfg.worldFogColor);
          scene.fog = cfg.worldFogMode === 'exponential'
            ? {isFogExp2:true, color, density:0.005, __amnesisFog:true, __amnesisMode:'exponential'}
            : {isFog:true, color, near:20, far:125, __amnesisFog:true, __amnesisMode:'distance'};
        } else if (!updateWorldVisuals.fogWarned) {
          console.warn('[AMNESIS World] Unable to derive renderer Color for native fog.');
          updateWorldVisuals.fogWarned = true;
        }
      }
      if (scene.fog?.__amnesisFog) {
        const fog = scene.fog;
        fog.color.set(cfg.worldFogColor);
        if (fog.isFog) {
          fog.near = Number(cfg.worldFogNear);
          fog.far = Math.max(fog.near + 1, Number(cfg.worldFogFar));
        } else {
          // Exponential density is in inverse world units, lower is gentler.
          const percent = Number(cfg.worldFogDensity) / 100;
          fog.density = 0.0001 + 0.045 * percent * percent;
        }
      }
    } else if (scene.fog?.__amnesisFog || scene.fog !== original.fog) {
      scene.fog = original.fog;
    }
  }
  // GPU animated world-space rain. The falling motion happens in the vertex
  // shader: no CPU geometry uploads or static point sprites are involved.
  // Weather collision uses the game's world geometry, not a screen overlay.
  const rainFX = {lib:null, loading:false, failed:false, scene:null,
    lines:null, geo:null, mat:null, count:0, fall:0, last:0, scanAt:0,
    tex:null, data:null, cursor:0, cellX:null, cellZ:null,
    scanOrder:null, centerX:null, centerZ:null, warnedNoPhysics:false};
  const RAIN_GRID = 64, RAIN_CELL = 0.5, RAIN_SCANS = 64;
  const RAIN_SCAN_INTERVAL = 50;
  // The rain is born 20 world units above the local player, and ends
  // eight units below them. Roofing/ground scans begin within this column.
  const RAIN_TOP = 20, RAIN_BOTTOM = -8;
  const RAIN_COLUMN = RAIN_TOP - RAIN_BOTTOM;

  async function rainLibrary(g) {
    if (rainFX.lib || rainFX.loading || rainFX.failed) return;
    const t = window.THREE || g?.THREE;
    if (t?.ShaderMaterial && t?.LineSegments && t?.BufferGeometry &&
        t?.Float32BufferAttribute) {rainFX.lib=t;return;}
    rainFX.loading=true;
    try {
      rainFX.lib=await import('https://cdn.jsdelivr.net/npm/three@0.160.1/build/three.module.js');
    } catch(error) {
      rainFX.failed=true;
      console.warn('[AMNESIS Rain] Unable to load Three.js:',error);
    } finally {rainFX.loading=false;}
  }
  function rainDispose() {
    rainFX.lines?.parent?.remove(rainFX.lines);
    rainFX.geo?.dispose?.();
    rainFX.mat?.dispose?.();
    rainFX.tex?.dispose?.();
    Object.assign(rainFX, {scene:null,lines:null,geo:null,mat:null,count:0,
      tex:null,data:null,cursor:0,cellX:null,cellZ:null,scanOrder:null,
      centerX:null,centerZ:null,last:0,scanAt:0});
  }

  // Build a center-out scan order so rain appears around the player first.
  function rainScanOrder() {
    const offsets=[];
    for(let dz=-RAIN_GRID/2; dz<RAIN_GRID/2; dz++) {
      for(let dx=-RAIN_GRID/2; dx<RAIN_GRID/2; dx++) {
        offsets.push({dx,dz,r2:dx*dx+dz*dz});
      }
    }
    offsets.sort((a,b)=>a.r2-b.r2 || a.dz-b.dz || a.dx-b.dx);
    return offsets;
  }

  function rainCreate(g) {
    const T=rainFX.lib;
    if(!T || !g?.scene) return;
    const intensity=Math.max(0,Math.min(100,Number(cfg.worldRainIntensity)||0))/100;
    const count=Math.round(220+2000*Math.pow(intensity,1.2));
    if(rainFX.lines && rainFX.scene===g.scene && rainFX.count===count) return;
    rainDispose();
    if(!intensity) return;
    const positions=new Float32Array(count*6);
    const tips=new Float32Array(count*2);
    for(let i=0;i<count;i++) {
      const a=i*6, k=i*2;
      // Distribute the rain seeds within the 28-unit player-relative column.
      const x=(Math.random()-.5)*32, y=Math.random()*RAIN_COLUMN, z=(Math.random()-.5)*32;
      positions[a]=x; positions[a+1]=y; positions[a+2]=z;
      positions[a+3]=x; positions[a+4]=y; positions[a+5]=z;
      tips[k]=0; tips[k+1]=1;
    }
    // Packed height RG, valid B. Unknown cells start hidden to prevent
    // rain showing through unscanned building roofs. 1/64m precision.
    // height = (R*256 + G)/64 - 256; B=255 is scanned/valid.
    const total=RAIN_GRID*RAIN_GRID;
    const data=new Uint8Array(total*4);
    for(let i=0;i<total;i++) data[i*4+3]=255;
    const tex=new T.DataTexture(data,RAIN_GRID,RAIN_GRID);
    tex.magFilter=T.NearestFilter; tex.minFilter=T.NearestFilter;
    tex.generateMipmaps=false; tex.flipY=false; tex.needsUpdate=true;

    const geometry=new T.BufferGeometry();
    geometry.setAttribute('position',new T.Float32BufferAttribute(positions,3));
    geometry.setAttribute('aTip',new T.Float32BufferAttribute(tips,1));
    const material=new T.ShaderMaterial({
      uniforms:{
        uOrigin:{value:new T.Vector3()},
        uFall:{value:0},
        uLength:{value:0.8},
        uRainTop:{value:0},
        uRainBottom:{value:0},
        uCeil:{value:tex},
        uColor:{value:new T.Color(cfg.worldRainColor)},
        uOpacity:{value:clamp01(Number(cfg.worldRainOpacity) / 100)}
      },
      vertexShader:`
        attribute float aTip;
        uniform vec3 uOrigin;
        uniform float uFall;
        uniform float uLength;
        uniform float uRainTop;
        uniform float uRainBottom;
        uniform sampler2D uCeil;
        varying float vTip;
        varying float vFade;
        void main() {
          vTip = aTip;
          vec3 p = position;
          p.x = uOrigin.x + mod(position.x - uOrigin.x + 16.0, 32.0) - 16.0;
          p.z = uOrigin.z + mod(position.z - uOrigin.z + 16.0, 32.0) - 16.0;
          // Spawn from player Y + 20, falling to player Y - 8.
          float column = max(uRainTop - uRainBottom, 1.0);
          float h = mod(position.y - uFall - uRainBottom, column);
          float yTop = uRainBottom + h;
          p.y = yTop - aTip * uLength;
          float d = length(p.xz - uOrigin.xz) / 16.0;
          float edge = min(2.0, column * 0.15);
          vFade = smoothstep(0.0, edge, h) * (1.0 - smoothstep(column - edge, column, h))
                * (1.0 - smoothstep(0.7, 1.0, d));
          // Sample the same world cell for both endpoints. Data is a circular
          // world-coordinate grid; never based on screen/camera position.
          vec2 cell = mod(floor(p.xz / ${RAIN_CELL.toFixed(2)}), ${RAIN_GRID.toFixed(1)});
          vec4 t = texture2D(uCeil, (cell + 0.5) / ${RAIN_GRID.toFixed(1)});
          if (t.b < 0.5) {
            // Until this cell is scanned, don't allow roof leakage.
            vFade = 0.0;
          } else {
            float hi = floor(t.r * 255.0 + 0.5);
            float lo = floor(t.g * 255.0 + 0.5);
            // First solid surface encountered by rain from above.
            float stopY = (hi * 256.0 + lo) / 64.0 - 256.0 + 0.06;
            if (yTop <= stopY) vFade = 0.0;
            p.y = max(p.y, stopY);
          }
          gl_Position = projectionMatrix * viewMatrix * vec4(p, 1.0);
        }`,
      fragmentShader:`
        uniform vec3 uColor;
        uniform float uOpacity;
        varying float vTip;
        varying float vFade;
        void main() {
          if (vFade <= 0.001) discard;
          gl_FragColor = vec4(uColor, uOpacity * vFade * (1.0 - 0.35 * vTip));
        }`,
      transparent:true, depthTest:true, depthWrite:false
    });
    const lines=new T.LineSegments(geometry,material);
    lines.frustumCulled=false;
    lines.renderOrder=5;
    g.scene.add(lines);
    const cellX=new Int32Array(total),cellZ=new Int32Array(total);
    cellX.fill(-2147483648);cellZ.fill(-2147483648);
    Object.assign(rainFX,{scene:g.scene,lines,geo:geometry,mat:material,count,
      tex,data,cellX,cellZ,cursor:0,centerX:null,centerZ:null,
      scanOrder:rainScanOrder()});
    console.info('[AMNESIS Rain] Player-relative world rain active:',count,'streaks; spawn Y = player Y + 20');
  }

  // Trace rain against visible, upward-facing bullet geometry, not player
  // clips/sky shell brushes. Continue beyond skipped obstructions.
  const RAIN_TRACE_MASK = 2;      // Bullet collision: world surfaces.
  const RAIN_MIN_FACING = 0.35;   // Upward-facing surface threshold.
  const RAIN_TRACE_STEPS = 6;     // Maximum rejected hits per column.
  const RAIN_NONVISUAL =
    /(^|[^a-z])(clip|sky|skybox|nodraw|blockbullets?|trigger|hint|skip|areaportal|portal|occluder|invisible|bounds|barrier)([^a-z]|$)/i;
  const rainSkippedMats = new Set();

  function rainIsNonVisual(mat) {
    const name = String(mat || '');
    if (!name || !RAIN_NONVISUAL.test(name)) return false;
    if (!rainSkippedMats.has(name)) {
      rainSkippedMats.add(name);
      console.info('[AMNESIS Rain] Ignoring collision-only rain surface:', name);
    }
    return true;
  }

  // First visible upward-facing surface between top and bottom, or null.
  function rainSurfaceY(physics, x, z, top, bottom) {
    let from = top;
    for (let step = 0; step < RAIN_TRACE_STEPS; step++) {
      const span = from - bottom;
      if (!(span > 0.02)) return null;
      const hit = physics.raycast(x, from, z, 0, -1, 0, span, RAIN_TRACE_MASK);
      if (!hit || !Number.isFinite(hit.t) || hit.t < 0 || hit.t > span) return null;
      const y = from - hit.t;
      const facing = Number.isFinite(hit.ny) ? hit.ny : 1;
      const mat = hit.mat || hit.b?.mat;
      if (facing >= RAIN_MIN_FACING && !rainIsNonVisual(mat)) return y;
      // The engine skips a box/hull when a ray starts inside it.
      from = y - 0.02;
    }
    return null;
  }

  // Scan downward from player Y + 20, using the first collision as the
  // rain cutoff (roof, overhang, ground). Unknown cells stay hidden.
  // Collision-only surfaces and side/back faces are intentionally skipped.
  function rainScanCeiling(g, player) {
    const physics=g.physics;
    if(!rainFX.data || typeof physics?.raycast!=='function') return;
    const px=Number(player.x)||0, py=Number(player.y)||0, pz=Number(player.z)||0;
    const cx=Math.floor(px/RAIN_CELL), cz=Math.floor(pz/RAIN_CELL);
    const half=RAIN_GRID/2,total=RAIN_GRID*RAIN_GRID;
    const wrap=v=>((v%RAIN_GRID)+RAIN_GRID)%RAIN_GRID;
    let changed=false;
    if(cx!==rainFX.centerX || cz!==rainFX.centerZ) {
      for(let j=0;j<total;j++) {
        const x=rainFX.cellX[j], z=rainFX.cellZ[j];
        if(x<cx-half || x>=cx+half || z<cz-half || z>=cz+half) {
          if(rainFX.data[j*4+2]) {rainFX.data[j*4+2]=0;changed=true;}
          rainFX.cellX[j]=-2147483648;
          rainFX.cellZ[j]=-2147483648;
        }
      }
      rainFX.centerX=cx;rainFX.centerZ=cz;
    }
    const top=py+RAIN_TOP-0.08;
    const bottom=py+RAIN_BOTTOM-5;
    const encode=y=>Math.round(Math.max(0,Math.min(65535,(y+256)*64)));
    const scan=(dx,dz)=>{
      const wx=cx+dx,wz=cz+dz;
      const i=wrap(wx),j=wrap(wz);
      const index=j*RAIN_GRID+i,o=index*4;
      if(rainFX.cellX[index]!==wx || rainFX.cellZ[index]!==wz) rainFX.data[o+2]=0;
      const x=(wx+0.5)*RAIN_CELL,z=(wz+0.5)*RAIN_CELL;
      let cutoff=bottom-1,valid=false;
      try {
        const surface=rainSurfaceY(physics,x,z,top,bottom);
        if(surface!==null) cutoff=surface;
        valid=true;
      } catch(error) {
        if(!rainFX.warnedNoPhysics) {
          rainFX.warnedNoPhysics=true;
          console.warn('[AMNESIS Rain] Surface collision query failed:',error);
        }
      }
      if(valid) {
        const h=encode(cutoff);
        rainFX.data[o]=h>>8;rainFX.data[o+1]=h&255;rainFX.data[o+2]=255;
        rainFX.cellX[index]=wx;rainFX.cellZ[index]=wz;
      }
      changed=true;
    };
    // Keep local cover responsive before the staggered full-area scan.
    for(let dz=-1;dz<=1;dz++) for(let dx=-1;dx<=1;dx++) scan(dx,dz);
    for(let n=0;n<RAIN_SCANS;n++) {
      const next=rainFX.scanOrder[rainFX.cursor++];
      if(rainFX.cursor>=total) rainFX.cursor=0;
      scan(next.dx,next.dz);
    }
    if(changed) rainFX.tex.needsUpdate=true;
  }

  function updateWorldRain(g) {
    if(!cfg.worldEnabled || !cfg.worldRain || !g?.scene || !g?.player ||
        Number(cfg.worldRainIntensity)<=0) {
      if(rainFX.lines) rainDispose();
      rainFX.last=0;
      rainFX.scanAt=0;
      return;
    }
    if(!rainFX.lib) {rainLibrary(g);return;}
    rainCreate(g);
    if(!rainFX.lines) return;
    const player=g.player;
    const speed=Math.max(10,Math.min(100,Number(cfg.worldRainSpeed)||65));
    const now=performance.now();
    const dt=rainFX.last ? Math.min(0.1,(now-rainFX.last)/1000) : 0;
    rainFX.last=now;
    const u=rainFX.mat.uniforms;
    rainFX.lines.visible=true;
    u.uColor.value.set(cfg.worldRainColor);
    u.uOpacity.value=clamp01(Number(cfg.worldRainOpacity) / 100);
    const py=Number(player.y)||0;
    const bottom=py+RAIN_BOTTOM;
    const top=py+RAIN_TOP;
    const column=RAIN_COLUMN;
    // CPU double-precision accumulation; wrap by the active column length.
    // Slider changes never reset the fall distance.
    rainFX.fall=(rainFX.fall+dt*(20+speed*0.6))%column;
    u.uFall.value=rainFX.fall;
    u.uOrigin.value.set(Number(player.x)||0,Number(player.y)||0,Number(player.z)||0);
    u.uRainTop.value=top;
    u.uRainBottom.value=bottom;
    u.uLength.value=0.45+speed*0.013;
    rainFX.lines.position.set(0,0,0);
    if(now-rainFX.scanAt>=RAIN_SCAN_INTERVAL) {
      rainFX.scanAt=now;
      rainScanCeiling(g,player);
    }
  }

  const runtimeFovOriginals = new WeakMap();
  const runtimeTracerOriginals = new WeakMap();
  const runtimeHitmarkerOriginals = new WeakMap();
  const skinStateBackups = new Map();
  const skinLastSignatures = new WeakMap();
  const skinRecords = new Map();
  let skinAppliedState = null;
  let randomKnifeModel=null,randomGloveModel=null,skinRandomNonce=0;
  let watermarkEl = null;
  let keybindListEl = null;
  let sessionStatsEl = null;
  let infoHudEl = null, vignetteEl = null;
  let fpsFrames=0,fpsValue=0,fpsLast=performance.now();
  const saturationOriginals=new WeakMap();

  const pickDifferent=(items,current)=>{
    const choices=items.filter(item=>item!==current);
    return choices[Math.floor(Math.random()*choices.length)]||items[0];
  };
  const effectiveKnifeModel=()=>cfg.skinRandomKnife?
    (randomKnifeModel||=pickDifferent(SKIN_KNIFE_MODELS,null)):cfg.skinKnifeModel;
  const effectiveGloveModel=()=>cfg.skinBareHands?'default':cfg.skinRandomGloves?
    (randomGloveModel||=pickDifferent(SKIN_GLOVE_MODELS,null)):cfg.skinGloveModel;
  function rerollRuntimeSkins() {
    if (cfg.skinRandomKnife) randomKnifeModel=pickDifferent(SKIN_KNIFE_MODELS,randomKnifeModel);
    if (cfg.skinRandomGloves) randomGloveModel=pickDifferent(SKIN_GLOVE_MODELS,randomGloveModel);
    skinRandomNonce++;
  }

  function skinRecord(model) {
    if (!skinRecords.has(model)) skinRecords.set(model, {
      id:'amnesis_'+model,name:titlecase(model.replace(/^knife_/,'')),
      weapon:model,slot:model,kind:'knife',kit:'',vanilla:true,
      rarity:'gold',wmin:0,wmax:0,phase:'',seed:0
    });
    return skinRecords.get(model);
  }

  function backupSkinState(state) {
    if (skinStateBackups.has(state)) return;
    const backup={};
    for (const key of ['skinOverride','skinItem','shownSkin','shownSet']) {
      backup[key]={had:Object.prototype.hasOwnProperty.call(state,key),value:state[key]};
    }
    skinStateBackups.set(state,backup);
  }

  function restoreSkinStates() {
    for (const [state,backup] of skinStateBackups) {
      for (const [key,data] of Object.entries(backup)) {
        if (data.had) state[key]=data.value;
        else delete state[key];
      }
    }
    skinStateBackups.clear();
    skinAppliedState=null;
  }

  function installSkinChangerHooks(g) {
    if (g.__amnesisSkinHook || typeof g._povLoadout!=='function' ||
        typeof g._ownGloves!=='function') return;
    const originalPov=g._povLoadout;
    const originalGloves=g._ownGloves;
    g._povLoadout=function(entity,...args) {
      const base=originalPov.call(this,entity,...args);
      if (!cfg.skinChanger||(entity&&entity!==this.player)||!base) return base;
      const out={...base,paintOpts:{...(base.paintOpts||{})}};
      const logical=this.weapons?.current;
      const knifeModel=effectiveKnifeModel();
      if (cfg.skinCurrentMode==='default') {
        delete out.paintOpts.kit;delete out.paintOpts.item;
      }
      if (logical==='knife') {
        if (knifeModel==='default') {
          out.wid=this.player?.team==='T'?'knife_t':'knife_ct';
        } else if (SKIN_KNIFE_MODELS.includes(knifeModel)) {
          out.wid=knifeModel;
        }
      }
      return out;
    };
    g._ownGloves=function(...args) {
      const gloveModel=effectiveGloveModel();
      if (!cfg.skinChanger||gloveModel==='native') {
        return originalGloves.apply(this,args);
      }
      if (gloveModel==='default') return null;
      return {model:gloveModel,kit:'',item:{wear:0,seed:0}};
    };
    g.__amnesisSkinHook={originalPov,originalGloves};
  }

  function refreshSkinChanger(g) {
    g._kx7Weapon=null;g._kx7Paint=null;
    const current=g.weapons?.current;
    if (current) g.viewmodel?.setWeapon?.(current);
    g.syncOwnGloves?.();
    g.hud?.updateWeapon?.();
    if (typeof tp!=='undefined') {
      tp.weapon=null;tp.paint=null;tp.glove=null;
    }
  }

  function updateRuntimeSkinChanger(g) {
    installSkinChangerHooks(g);
    const current=g.weapons?.current;
    const state=current?g.weapons?.state?.(current):null;
    const knifeModel=effectiveKnifeModel();
    const gloveModel=effectiveGloveModel();
    const signature=[cfg.skinChanger,cfg.skinCurrentMode,
      knifeModel,gloveModel,current,skinRandomNonce].join('|');
    const needsState=cfg.skinChanger&&
      (cfg.skinCurrentMode==='default'||
       (current==='knife'&&SKIN_KNIFE_MODELS.includes(knifeModel)));
    if (skinLastSignatures.get(g)===signature&&
        (!needsState||skinAppliedState===state)) return;
    restoreSkinStates();
    if (needsState&&state) {
      backupSkinState(state);
      if (current==='knife'&&SKIN_KNIFE_MODELS.includes(knifeModel)) {
        const skin=skinRecord(knifeModel);
        state.skinOverride=skin;state.skinItem={wear:0,seed:0};
        state.shownSkin=skin;state.shownSet=true;
      } else {
        state.skinOverride='default';state.skinItem=null;
        state.shownSkin=null;state.shownSet=true;
      }
      skinAppliedState=state;
    }
    skinLastSignatures.set(g,signature);
    refreshSkinChanger(g);
  }

  function updateRuntimeFeatures(g) {
    updateRuntimeSkinChanger(g);
    fpsFrames++;
    const fpsNow=performance.now();
    if (fpsNow-fpsLast>=500) {
      fpsValue=Math.round(fpsFrames*1000/(fpsNow-fpsLast));
      fpsFrames=0;fpsLast=fpsNow;
    }
    if (cfg.localTracers) {
      if (!runtimeTracerOriginals.has(g)) runtimeTracerOriginals.set(g, g.fpTracers);
      g.fpTracers = true;
    } else if (runtimeTracerOriginals.has(g)) {
      g.fpTracers = runtimeTracerOriginals.get(g);
      runtimeTracerOriginals.delete(g);
    }

    const xh = g.hud?.xh;
    if (xh && cfg.forceHitmarkers) {
      if (!runtimeHitmarkerOriginals.has(xh)) runtimeHitmarkerOriginals.set(xh, xh.hit);
      xh.hit = 1;
    } else if (xh && runtimeHitmarkerOriginals.has(xh)) {
      xh.hit = runtimeHitmarkerOriginals.get(xh);
      runtimeHitmarkerOriginals.delete(xh);
    }

    const camera = g.camera;
    if (camera && cfg.fovOverride) {
      if (!runtimeFovOriginals.has(g)) runtimeFovOriginals.set(g, g.baseFov);
      const fov = Math.max(60, Math.min(130, Number(cfg.cameraFov) || 90));
      g.baseFov = fov;
      if (!g.scoped && camera.fov !== fov) {
        camera.fov = fov;
        camera.updateProjectionMatrix?.();
      }
    } else if (camera && runtimeFovOriginals.has(g)) {
      const fov = runtimeFovOriginals.get(g);
      g.baseFov = fov;
      if (!g.scoped) {
        camera.fov = fov;
        camera.updateProjectionMatrix?.();
      }
      runtimeFovOriginals.delete(g);
    }

    if (cfg.removeFlash && g.player) {
      const p = g.player;
      p.flashStart = 0; p.flashDur = 0; p.flashUntil = 0;
      p.flashBlindUntil = 0; p.flashRamp = false;
      g._flashSnap = null;
      if (g.hud?.flashEl) g.hud.flashEl.style.opacity = 0;
      if (g.hud?.flashImg) g.hud.flashImg.style.opacity = 0;
    }

    const smokeOv = g.hud?.smokeOv;
    if (smokeOv) smokeOv.style.visibility = cfg.noSmoke ? 'hidden' : '';
    if (cfg.noSmoke && Array.isArray(g.grenadeMgr?.smokes)) {
      for (const smoke of g.grenadeMgr.smokes) {
        smoke.handle?.stop?.();
        if (smoke.cloud) g.grenadeMgr._releaseCloud?.(smoke.cloud);
        smoke.cloud = null; smoke.sprites = null; smoke.job = null;
        smoke.until = g.time;
      }
    }

    const scopeEl = g.hud?.scopeEl;
    if (scopeEl) scopeEl.style.visibility = cfg.noScopeOverlay ? 'hidden' : '';

    const hudRoot=g.hud?.root;
    if (hudRoot) {
      const menuOpen=hudRoot.classList?.contains('inmenu')||g.hud?.pauseMode||g.hud?.buyOpen;
      hudRoot.style.visibility=cfg.hideNativeHud&&g.state==='playing'&&!menuOpen?'hidden':'';
    }

    if (pendingInspect) {
      const weapons=g.weapons,now=performance.now();
      if (!cfg.autoInspectKill||now>pendingInspect.expires||!g.player?.alive||
          weapons?.current!==pendingInspect.weapon) pendingInspect=null;
      else if (!weapons.reloading&&(Number(weapons.drawT)||0)<=0&&
          !(Number(weapons.scopeLevel)>0)&&
          !weapons.grenadeCooking&&!weapons.charging&&
          g.time>=(weapons.nextFire||0)&&!g.viewmodel?.action) {
        g.viewmodel?.inspect?.();
        g.kx7?.inspect?.({deploying:false});
        pendingInspect=null;
      }
    }

    if (cfg.watermark) {
      if (!watermarkEl) {
        watermarkEl = document.createElement('div');
        watermarkEl.id = 'amnesis-watermark';
        watermarkEl.style.cssText =
          'position:fixed;top:12px;right:12px;z-index:2147483644;pointer-events:none;' +
          'padding:7px 10px;background:#101215e8;border:1px solid #30343a;' +
          'border-top:2px solid #ff3b4a;color:#f2f3f5;font:700 10px Consolas,monospace;' +
          'letter-spacing:1px;box-shadow:0 8px 24px #0008';
        document.documentElement.append(watermarkEl);
      }
      const state = g.state === 'playing' ? 'ACTIVE' : 'IDLE';
      watermarkEl.textContent = 'AMNESIS  /  ' + state;
      watermarkEl.style.borderTopColor = cfg.accent;
      watermarkEl.style.display = 'block';
    } else if (watermarkEl) {
      watermarkEl.remove();
      watermarkEl = null;
    }

    if (cfg.keybindList) {
      if (!keybindListEl) {
        keybindListEl = document.createElement('div');
        keybindListEl.id = 'amnesis-keybinds';
        keybindListEl.style.cssText =
          'position:fixed;top:50px;right:12px;z-index:2147483644;pointer-events:none;' +
          'min-width:150px;padding:8px 10px;background:#101215e8;border:1px solid #30343a;' +
          'color:#dfe2e7;font:700 10px Consolas,monospace;line-height:1.65;' +
          'white-space:pre;box-shadow:0 8px 24px #0008';
        document.documentElement.append(keybindListEl);
      }
      const active = Object.entries(binds)
        .filter(([key]) => cfg[key] === true)
        .map(([key, code]) => key.toUpperCase() + '  [' + keyName(code) + ']');
      keybindListEl.textContent = active.length ? active.join('\n') : 'NO ACTIVE BINDS';
      keybindListEl.style.top = cfg.watermark ? '52px' : '12px';
      keybindListEl.style.borderTop = '2px solid ' + cfg.accent;
    } else if (keybindListEl) {
      keybindListEl.remove();
      keybindListEl = null;
    }

    if (cfg.killCounter||cfg.damageCounter) {
      if (!sessionStatsEl) {
        sessionStatsEl=document.createElement('div');
        sessionStatsEl.id='amnesis-session-stats';
        sessionStatsEl.style.cssText=
          'position:fixed;left:50%;top:78px;transform:translateX(-50%);' +
          'z-index:2147483644;pointer-events:none;padding:6px 12px;' +
          'background:#101215d9;border:1px solid #30343a;color:#f2f3f5;' +
          'font:700 11px Consolas,monospace;letter-spacing:1px;box-shadow:0 6px 20px #0008';
        document.documentElement.append(sessionStatsEl);
      }
      const parts=[];
      if (cfg.killCounter) parts.push('KILLS '+sessionStats.kills);
      if (cfg.damageCounter) parts.push('DAMAGE '+Math.round(sessionStats.damage));
      sessionStatsEl.textContent=parts.join('  /  ');
      sessionStatsEl.style.borderTop='2px solid '+cfg.accent;
    } else if (sessionStatsEl) {
      sessionStatsEl.remove();sessionStatsEl=null;
    }

    if (cfg.clockHud||cfg.fpsHud||cfg.coordinatesHud) {
      if (!infoHudEl) {
        infoHudEl=document.createElement('div');
        infoHudEl.id='amnesis-info-hud';
        infoHudEl.style.cssText=
          'position:fixed;left:14px;bottom:14px;z-index:2147483644;pointer-events:none;' +
          'padding:7px 10px;background:#101215d9;border:1px solid #30343a;' +
          'color:#f2f3f5;font:700 10px Consolas,monospace;line-height:1.6;' +
          'letter-spacing:.8px;white-space:pre;box-shadow:0 6px 20px #0008';
        document.documentElement.append(infoHudEl);
      }
      const lines=[];
      if (cfg.clockHud) lines.push(new Date().toLocaleTimeString());
      if (cfg.fpsHud) lines.push('FPS  '+fpsValue);
      if (cfg.coordinatesHud&&g.player) lines.push('XYZ  '+
        [g.player.x,g.player.y,g.player.z].map(v=>(Number(v)||0).toFixed(1)).join('  '));
      infoHudEl.textContent=lines.join('\n');
      infoHudEl.style.borderLeft='2px solid '+cfg.accent;
    } else if (infoHudEl) {infoHudEl.remove();infoHudEl=null;}

    if (cfg.screenVignette) {
      if (!vignetteEl) {
        vignetteEl=document.createElement('div');
        vignetteEl.id='amnesis-vignette';
        vignetteEl.style.cssText='position:fixed;inset:0;z-index:8;pointer-events:none';
        document.documentElement.append(vignetteEl);
      }
      const color=String(cfg.vignetteColor||'#000000');
      const alpha=Math.max(0,Math.min(100,Number(cfg.vignetteOpacity)||0))/100;
      vignetteEl.style.background=`radial-gradient(circle,transparent 42%,color-mix(in srgb,${color} ${alpha*100}%,transparent) 100%)`;
    } else if (vignetteEl) {vignetteEl.remove();vignetteEl=null;}

    const canvas=g.renderer?.domElement;
    if (canvas&&cfg.worldSaturation) {
      if (!saturationOriginals.has(canvas)) saturationOriginals.set(canvas,canvas.style.filter);
      canvas.style.filter=`saturate(${Math.max(0,Math.min(300,Number(cfg.saturationPercent)||100))}%)`;
    } else if (canvas&&saturationOriginals.has(canvas)) {
      canvas.style.filter=saturationOriginals.get(canvas);saturationOriginals.delete(canvas);
    }

    if (box) {
      box.classList.toggle('ui-blur',!!cfg.uiBlur);
      box.classList.toggle('no-ui-animation',!cfg.uiAnimations);
      box.style.opacity=cfg.uiTransparency?
        String(Math.max(.4,Math.min(1,(Number(cfg.uiOpacity)||92)/100))):'1';
    }
  }

  function visualSettingsFrame() {
    requestAnimationFrame(visualSettingsFrame);
    const g = window.game;
    if (!g) return;
    try {
      updateRuntimeFeatures(g);
      updateViewmodelChams(g); updateWorldVisuals(g); updateWorldRain(g);
    }
    catch (err) { console.warn('[AMNESIS Visual Settings]', err); }
  }
  requestAnimationFrame(visualSettingsFrame);

  // =====================================================
  // SILENT AIM
  // =====================================================

  let hookedGame = null;

  // =====================================================
  // RAGE BOT / HITBOX RESOLVER
  // =====================================================
  // This game's bullet engine intersects animated world-space capsules,
  // NOT the rendered silhouette or CS:GO-style network desync angles.
  // These definitions mirror the updated game's ci(bones) hitbox profile.
  // The resolver follows the SAME collision capsules fireBullet uses.
  const rageProfiles = [
    ['head_0',       4.3, [-1,1.8,0],      [3.5,.2,0],      'head'],
    ['neck_0',       3.5, [0,-.4,0],       [1.4,-.2,0],     'neck'],
    ['spine_1',      6.5, [3.8,.8,-2.4],   [3.8,.4,2.4],    'chest'],
    ['spine_2',      6.2, [4.8,.15,-4.1],  [4.8,.15,4.1],   'chest'],
    ['spine_3',      5.0, [2.5,-.6,-6],    [2.5,-.6,6],     'chest'],
    ['spine_0',      6.0, [1.4,.8,3.1],    [1.4,.8,-3.1],   'stomach'],
    ['pelvis',       6.0, [-2.7,1.1,-3.2],[-2.7,1.1,3.2], 'stomach'],
    ['arm_upper_l',  3.3, [0,0,0],          [11.2,0,0],      'arms'],
    ['arm_lower_l',  3.0, [0,0,0],          [10,0,0],        'arms'],
    ['hand_l',       2.3, [0,.3,0],         [3.59,1.15,.11], 'arms'],
    ['arm_upper_r',  3.3, [0,0,0],          [-11.2,0,0],     'arms'],
    ['arm_lower_r',  3.0, [0,0,0],          [-10,0,-.5],     'arms'],
    ['hand_r',       2.3, [0,-.3,.02],      [-3.44,-1.17,-.09], 'arms'],
    ['leg_upper_l',  5.0, [1.3,-.2,0],      [16.5,-.7,0],    'legs'],
    ['leg_lower_l',  4.0, [.1,-.4,.2],      [17,-.4,.7],     'legs'],
    ['ankle_l',      2.6, [0,-3.43,-.52],   [8,.74,.33],     'legs'],
    ['leg_upper_r',  5.0, [-1.3,0,-.6],     [-16.5,0,-.7],   'legs'],
    ['leg_lower_r',  4.0, [-.1,0,-.2],      [-17,.4,-.7],    'legs'],
    ['ankle_r',      2.6, [-7.98,-.75,-.27],[-.02,3.44,.58], 'legs']
  ];
  const rageRigProfiles = new WeakMap();
  let rageLastTarget = null;
  let rageLockedEnemy = null;
  let rageHookedWeapons = null;
  let rageStateGame=null,rageRoundToken=null;

  function rageCombatActive(g,requireInput=true) {
    if (!g||g.state!=='playing'||g.roundActive!==true||!g.player?.alive||
        g.frozen||g._joining||document.hidden) return false;
    if (g.modeCtl&&!['live','planted'].includes(g.modeCtl.state)) return false;
    if (requireInput&&(uiOpen||!g.input?.locked||g.hud?.pauseMode||
        g.hud?.buyOpen||g.hud?.howToOpen)) return false;
    return true;
  }

  function resetRageTransient() {
    rageLastTarget=null;rageLockedEnemy=null;
  }

  function syncRageLifecycle(g) {
    const token=g?.modeCtl?.roundNum??g?.modeCtl?.round??
      (g?.roundActive?1:0);
    if (g!==rageStateGame||token!==rageRoundToken) {
      resetRageTransient();rageStateGame=g;rageRoundToken=token;
    }
    if (!cfg.rageBot||!rageCombatActive(g,false)) resetRageTransient();
    if (rageLockedEnemy&&(!rageLockedEnemy.alive||
        rageLockedEnemy.team===g?.player?.team)) rageLockedEnemy=null;
  }

  function forcedFireInput(input) {
    return new Proxy(input,{get(target,key,receiver) {
      if (key==='mouse1'||key==='mouse1Pressed') return true;
      return Reflect.get(target,key,receiver);
    }});
  }

  function rageWorldPoint(m, p) {
    return {
      x: m[0]*p[0]+m[4]*p[1]+m[8]*p[2]+m[12],
      y: m[1]*p[0]+m[5]*p[1]+m[9]*p[2]+m[13],
      z: m[2]*p[0]+m[6]*p[1]+m[10]*p[2]+m[14]
    };
  }

  function rageBuildCapsules(agent) {
    // The native hitboxes are initialized lazily by the bullet engine.
    // Before the first shot, build an equivalent subset for targeting,
    // without modifying the engine's own internal hitbox collection.
    if (agent?._hitboxes?.length) return agent._hitboxes;
    const bones = agent?.bones;
    if (!bones) return [];
    let boxes = rageRigProfiles.get(agent);
    if (!boxes) {
      const normalized = new Map(Object.entries(bones).map(
        ([name, bone]) => [name.toLowerCase(), bone]
      ));
      boxes = rageProfiles.map(([name, size, p0, p1, region]) => ({
        bone: bones[name] || normalized.get(name),
        p0, p1, r: size * 0.0254, region
      })).filter(item => item.bone);
      rageRigProfiles.set(agent, boxes);
    }
    return boxes;
  }

  // =====================================================
  // NATIVE WALLBANG / BREAKABLE PATH TEST
  // =====================================================
  // This reproduces main.js fireBullet world-collision rules:
  //  - bullet mask 2; 4 world penetrations maximum
  //  - penetration inches = weapon.penetration * 30
  //  - each material scales max thickness AND remaining damage
  //  - physics.traceExit uses a 4-inch step, so mirror it exactly
  //  - triangles stop bullets, though damageable map entities still take damage
  //  - blocked paths are rejected, including destructible map entities
  // Copying the material table makes this independent of game bundle minification.
  const rageMaterialFactors = {
    "armorflesh": [0.5, 0.3],
    "asphalt": [0.55, 0.3],
    "blockbullets": [0.01, 0.001],
    "brick": [0.47, 0.3],
    "cardboard": [0.95, 0.99],
    "carpet": [0.75, 0.5],
    "chainlink": [0.99, 0.99],
    "clay": [0.95, 0.6],
    "computer": [0.4, 0.45],
    "concrete": [0.5, 0.25],
    "default": [0.5, 0.5],
    "dirt": [0.6, 0.3],
    "flesh": [0.9, 0.5],
    "foliage": [0.95, 0.5],
    "fruit": [0.9, 0.5],
    "glass": [0.99, 0.5],
    "glassbottle": [0.99, 0.5],
    "glassfloor": [0.99, 0.5],
    "gravel": [0.4, 0.5],
    "ice": [0.75, 0.5],
    "metal": [0.4, 0.5],
    "metal_barrel": [0.01, 0.01],
    "metal_box": [0.5, 0.5],
    "metal_sand_barrel": [0.01, 0.01],
    "metaldogtags": [0.4, 0.5],
    "metalgrate": [0.95, 0.99],
    "metalpanel": [0.5, 0.45],
    "metalvehicle": [0.5, 0.5],
    "metalvent": [0.6, 0.45],
    "plaster": [0.7, 0.6],
    "plastic_barrel": [0.7, 0.5],
    "plastic_box": [0.75, 0.5],
    "plastic_solid": [0, 0.5],
    "player_control_clip": [0.5, 1],
    "porcelain": [0.95, 0.5],
    "pottery": [0.95, 0.6],
    "potterylarge": [0.95, 0.6],
    "quicksand": [0.2, 0.5],
    "rock": [0.5, 0.25],
    "rubber": [0.85, 0.5],
    "sand": [0.3, 0.25],
    "sheetrock": [0.85, 0.6],
    "snow": [0.85, 0.5],
    "solidmetal": [0.27, 0.3],
    "tile": [0.7, 0.3],
    "upholstery": [0.75, 0.5],
    "water": [0.3, 0.5],
    "watermelon": [0.95, 0.6],
    "wood": [0.9, 0.6],
    "wood_basket": [0.9, 0.5],
    "wood_box": [0.9, 0.5],
    "wood_crate": [0.9, 0.5],
    "wood_dense": [0.5, 0.3],
    "wood_ladder": [0.9, 0.5],
    "wood_plank": [0.85, 0.5],
    "wood_solid": [0.8, 0.5],
  };
  const rageMaterialAliases = {
    "crate": "wood_crate",
    "curb": "concrete",
    "fabric": "carpet",
    "paved": "asphalt",
    "plank": "wood_plank",
    "plaster2": "plaster",
    "plastic": "plastic_box",
    "tarp": "carpet",
  };
  const rageMaterialProperties = mat => {
    const key = String(mat ?? 'default').toLowerCase();
    return rageMaterialFactors[rageMaterialAliases[key] || key] ||
      rageMaterialFactors.default;
  };

  function rageBulletPath(g, origin, p) {
    const vx = p.x-origin.x, vy = p.y-origin.y, vz = p.z-origin.z;
    const dist = Math.hypot(vx,vy,vz);
    if (!(dist > 0.0001) || !Number.isFinite(dist)) return null;
    const physics = g.physics;
    if (typeof physics?.raycast !== 'function') return null;
    const dx=vx/dist, dy=vy/dist, dz=vz/dist;
    const pen=Math.max(0, Number(g.weapons?.def?.()?.penetration) || 0);
    const totalInches=pen*30;
    let remainingHits=totalInches>0 ? 4 : 0;
    let travelled=0, walls=0, damageScale=1;
    try {
      // 4 maximum penetrations; the extra pass validates the final free path.
      for (let pass=0;pass<=4;pass++) {
        const remaining=dist-travelled;
        if (remaining<=0.002) {
          return {mode:walls?'wallbang':'clear',walls,damageScale};
        }
        const sx=origin.x+dx*travelled;
        const sy=origin.y+dy*travelled;
        const sz=origin.z+dz*travelled;
        const hit=physics.raycast(sx,sy,sz,dx,dy,dz,remaining,2);
        if (!hit) return {mode:walls?'wallbang':'clear',walls,damageScale};
        if (!Number.isFinite(hit.t) || hit.t<0) return null;
        if (hit.t>=remaining-0.003) {
          return {mode:walls?'wallbang':'clear',walls,damageScale};
        }
        if (cfg.rageFullMapWallbang) {
          return {mode:'wallbang',walls:1,damageScale:1,bypassWorld:true};
        }
        // Snapshot all fields before traceExit: the physics engine reuses
        // its raycast hit object between queries.
        const impact=travelled+hit.t;
        const impactX=origin.x+dx*impact;
        const impactY=origin.y+dy*impact;
        const impactZ=origin.z+dz*impact;
        const material=hit.mat;
        const isTri=!!hit.b?.tri;
        // A blocked path is not an enemy hit, even if the obstacle has health.
        if (isTri || remainingHits<=0 || typeof physics.traceExit!=='function') {
          return null;
        }
        const [thicknessFactor,damageFactor]=rageMaterialProperties(material);
        const maxThickness=Math.min(90,totalInches*thicknessFactor)*0.0254;
        if (!(maxThickness>0)) {
          return null;
        }
        const exit=physics.traceExit(
          impactX,impactY,impactZ,dx,dy,dz,
          maxThickness,4*0.0254,2
        );
        if (!Number.isFinite(exit) || exit<0 ||
            impact+exit>=dist-0.005) {
          return null;
        }
        remainingHits--;
        walls++;
        damageScale*=damageFactor;
        travelled=impact+exit+0.002;
      }
    } catch (error) {
      console.warn('[Rage Bot] penetration check:', error);
    }
    return null;
  }

  function rageAimCandidate(g, origin, p, region, fwd, cosFov) {
    if (![p.x,p.y,p.z].every(Number.isFinite)) return null;
    const dx=p.x-origin.x, dy=p.y-origin.y, dz=p.z-origin.z;
    const length=Math.hypot(dx,dy,dz);
    if (length<0.05) return null;
    const dir={x:dx/length,y:dy/length,z:dz/length};
    const dot=dir.x*fwd.x+dir.y*fwd.y+dir.z*fwd.z;
    if (dot < cosFov-1e-7) return null;
    const path=rageBulletPath(g,origin,p);
    if (!path) return null;
    return {
      dir,point:p,length,angle:Math.acos(Math.max(-1,Math.min(1,dot))),
      region,path
    };
  }

  function rageTarget(g, origin) {
    syncRageLifecycle(g);
    if (!cfg.rageTargetLock) rageLockedEnemy=null;
    if (!cfg.rageBot || !rageCombatActive(g,false)) return null;
    const local=g.player;
    const pitch=Number(local.pitch), yaw=Number(local.yaw);
    if (!Number.isFinite(pitch)||!Number.isFinite(yaw)) return null;
    const cs=Math.cos(pitch);
    const fwd={x:-Math.sin(yaw)*cs,y:Math.sin(pitch),z:-Math.cos(yaw)*cs};
    const fov=Math.max(1,Math.min(180,Number(cfg.rageFov)||180));
    const cosFov=Math.cos(fov*Math.PI/180);
    const entities=characterEntities(g);
    let chosen=null, bestKey=null,lockedChosen=null,lockedKey=null;
    const betterKey=(a,b)=>!b||a.some((value,index)=>
      value!==b[index]&&a.slice(0,index).every((v,i)=>v===b[i])&&value<b[index]
    );

    for (const enemy of entities) {
      if (!enemy || enemy===local || !enemy.alive ||
          (local.team!=null && enemy.team===local.team) ||
          Number(enemy.spawnProt)>0) continue;
      if (![enemy.x,enemy.y,enemy.z].every(Number.isFinite)) continue;

      const agent=characterAgent(enemy);
      const possibilities=[];
      const bodyAim=cfg.rageBodyAimLowHp&&Number(local.health)<=
        Math.max(1,Math.min(100,Number(cfg.rageBodyHp)||35));
      const allowedRegions = cfg.rageForceHead ? new Set(['head']) :
        bodyAim?new Set(['chest','stomach']):new Set([
          ...(cfg.rageAreaHead ? ['head'] : []),
          ...(cfg.rageAreaNeckChest ? ['neck','chest'] : []),
          ...(cfg.rageAreaArms ? ['arms'] : []),
          ...(cfg.rageAreaTorso ? ['stomach'] : []),
          ...(cfg.rageAreaLegs ? ['legs'] : [])
        ]);
      if (!allowedRegions.size) continue;
      if (agent?.bones) {
        agent.root?.updateMatrixWorld?.(true);
        if (cfg.rageResolver) {
          // Convert the bullet engine's actual hitbox offsets into world
          // coordinates, accounting for animation and fake model rotations.
          for (const capsule of rageBuildCapsules(agent)) {
            const bone=capsule.bone;
            bone?.updateWorldMatrix?.(true,false);
            const m=bone?.matrixWorld?.elements;
            if (!m) continue;
            const a=rageWorldPoint(m,capsule.p0);
            const b=rageWorldPoint(m,capsule.p1);
            const region=capsule.region === 'arm' ? 'arms' :
              capsule.region === 'leg' ? 'legs' : capsule.region;
            if (!allowedRegions.has(region)) continue;
            // Center first; endpoint-biased samples support partial peeks.
            for (const t of [.5,.25,.75]) {
              possibilities.push({
                point:{x:a.x+(b.x-a.x)*t,y:a.y+(b.y-a.y)*t,z:a.z+(b.z-a.z)*t},
                region
              });
            }
          }
        } else {
          // Visual-only reference, deliberately NOT called a resolver.
          for (const [name,region] of [['head_0','head'],['neck_0','neck'],['spine_2','chest'],['pelvis','stomach'],['arm_upper_l','arms'],['arm_upper_r','arms'],['leg_upper_l','legs'],['leg_upper_r','legs']]) {
            if (!allowedRegions.has(region)) continue;
            const bone=agent.bones[name];
            bone?.updateWorldMatrix?.(true,false);
            const m=bone?.matrixWorld?.elements;
            if (m) possibilities.push({point:{x:m[12],y:m[13],z:m[14]},region});
          }
        }
      }
      // Game engine falls back to a standing cylinder when there is
      // no valid character rig. Don't invent a 'resolved' head in that case.
      if (!possibilities.length && !agent?.bones && allowedRegions.has('chest')) {
        const ey=Number(enemy._visY ?? enemy.y);
        const h=Number(enemy.height)||1.8;
        possibilities.push({point:{x:enemy.x,y:ey+h*.65,z:enemy.z},region:'chest'});
      }

      let candidate=null;
      const priority=(cfg.rageForceHead?['head']:
        ['head','neck','chest','arms','stomach','legs'])
        .filter(region => allowedRegions.has(region));
      const modes=['clear','wallbang'];
      for (const mode of modes) {
        for (const region of priority) {
          let bestRegion=null;
          for (const item of possibilities) {
            if (item.region!==region) continue;
            const hit=rageAimCandidate(g,origin,item.point,region,fwd,cosFov);
            if (!hit || hit.path.mode!==mode) continue;
            if (!bestRegion || hit.angle<bestRegion.angle) bestRegion=hit;
          }
          if (bestRegion) { candidate=bestRegion; break; }
        }
        if (candidate) break;
      }
      if (!candidate) continue;
      // Preserve visible-enemy priority: a target behind a wall near the
      // crosshair must not steal focus from a plainly visible opponent.
      // Only accept paths that reach an enemy; never shoot to clear obstacles.
      const modeRank={clear:0,wallbang:1}[candidate.path.mode];
      const primary=cfg.ragePriority==='distance'?candidate.length:
        cfg.ragePriority==='health'?Math.max(0,Number(enemy.health)||0):candidate.angle;
      const key=[modeRank,primary,candidate.angle,candidate.length,candidate.path.walls||0];
      if (betterKey(key,bestKey)) {
        bestKey=key;
        chosen={enemy,...candidate};
      }
      if (cfg.rageTargetLock&&enemy===rageLockedEnemy&&betterKey(key,lockedKey)) {
        lockedKey=key;lockedChosen={enemy,...candidate};
      }
    }
    if (lockedChosen) chosen=lockedChosen;
    if (cfg.rageTargetLock&&chosen) rageLockedEnemy=chosen.enemy;
    return chosen;
  }

  const firearmClasses = new Set(['pistol','rifle','smg','shotgun','sniper','mg']);
  let triggerTargetSince=0;

  function applyWeaponModifiers(weapons) {
    if (cfg.noRecoil) {
      const punch = weapons.punch;
      if (punch) {
        punch.aimPitch = 0; punch.aimYaw = 0;
        punch.velPitch = 0; punch.velYaw = 0;
        punch.viewPitch = 0; punch.viewYaw = 0;
      }
      weapons.punchP = 0; weapons.punchY = 0;
      weapons.shkP = 0; weapons.shkY = 0;
    }
    if (cfg.infiniteAmmo) {
      const def = weapons.def?.();
      const state = weapons.state?.();
      if (def && state && firearmClasses.has(def.class)) {
        if (Number.isFinite(def.mag) && def.mag > 0) state.ammo = def.mag;
        if (Number.isFinite(def.reserve)) {
          state.reserve = Math.max(Number(state.reserve) || 0, def.reserve);
        }
        weapons._autoReload = null;
      }
    }
  }

  function maybeAutoReload(weapons, game, input) {
    if (!cfg.autoReload || cfg.infiniteAmmo) return;
    const def = weapons.def?.();
    const state = weapons.state?.();
    if (!def || !state || !firearmClasses.has(def.class)) return;
    const mag = Number(def.mag);
    if (!(mag > 1)) return;
    const threshold = Math.max(0, Math.min(mag - 1,
      Math.floor(Number(cfg.autoReloadThreshold) || 0)));
    if (state.ammo > threshold || state.ammo >= mag || !(state.reserve > 0) ||
        weapons.reloading || weapons.drawT > 0 || weapons.grenadeCooking ||
        weapons.charging || (weapons._burstLeft || 0) > 0 || game.frozen ||
        game.time < (weapons.nextFire || 0) || input?.mouse1) return;
    weapons.startReload?.();
  }

  function ensureRageFireHook() {
    const g=window.game, w=g?.weapons;
    if (!g||!w || typeof w.update!=='function') return;
    if (w===rageHookedWeapons && w.update.__amnesisRageBot) return;
    const original=w.update;
    const wrapper=function(...args) {
      syncRageLifecycle(g);
      applyWeaponModifiers(this);
      const frameInput = args[1] || g.input;
      maybeAutoReload(this, g, frameInput);
      const runOriginal = (callArgs=args) => {
        applyWeaponModifiers(this);
        try { return original.apply(this,callArgs); }
        finally { applyWeaponModifiers(this); }
      };
      const runForcedFire=()=>{
        if (frameInput!==g.input) return runOriginal();
        const forcedArgs=args.slice();forcedArgs[1]=forcedFireInput(frameInput);
        const pullBefore=this._pull;let completed=false;
        try {
          this._pull=true;
          const result=runOriginal(forcedArgs);completed=true;return result;
        } finally {if(!completed)this._pull=pullBefore;}
      };
      rageLastTarget=null;

      // RAPID FIRE: bypass the game's nextFire/cycleTime cooldown for
      // firearms only. Retain its native ammo, draw, reload and shot code.
      // One shot is issued per game update while the fire button is held.
      const activeRapid = cfg.rapidFire && rageCombatActive(g,true) &&
        ['pistol', 'rifle', 'smg', 'shotgun', 'sniper', 'mg']
          .includes(this.def?.()?.class);
      if (activeRapid) {
        // Weapons.fire() schedules nextFire after every shot. The next
        // update clears that scheduled delay, but never creates ammo.
        this.nextFire = g.time;
        // Semi-auto guns ordinarily require a fresh mouse press each shot.
        // Holding fire should keep cycling without synthetic DOM events.
        if (frameInput?.mouse1 && !this.reloading && !this.charging) {
          this._pull = true;
        }
      }

      let triggerOnTarget=false;
      if (cfg.triggerBot && (!cfg.rageBot || !cfg.rageAutoFire) &&
          rageCombatActive(g,true) && frameInput===g.input) {
        const def = this.def?.();
        const ammo = this.state?.();
        const p = g.player;
        const origin = {x:p.x,y:p.y+Number(p.eyeH??1.55)-(p._stepLift||0),z:p.z};
        triggerOnTarget=!!(def && firearmClasses.has(def.class) && ammo?.ammo > 0 &&
            this.drawT <= 0 && !this.reloading && !this.charging &&
            g.time >= (this.nextFire || 0) && triggerTarget(g, origin));
        if (triggerOnTarget) {
          if (!triggerTargetSince) triggerTargetSince=performance.now();
          const delay=cfg.triggerDelay?Math.max(0,Math.min(500,
            Number(cfg.triggerDelayMs)||0)):0;
          if (performance.now()-triggerTargetSince<delay) return runOriginal();
          return runForcedFire();
        }
      }
      if (!triggerOnTarget) triggerTargetSince=0;

      if (!cfg.rageBot || !cfg.rageAutoFire ||
          !rageCombatActive(g,true) || frameInput!==g.input) {
        return runOriginal();
      }
      const def=this.def?.();
      const ammo=this.state?.();
      // Let the native fire/reload scheduler handle cadence and ammo;
      // do not trigger grenade throws, knife swings, or objective actions.
      if (!def || ['knife','grenade','c4','zeus'].includes(def.class) ||
          !ammo || !(ammo.ammo>0) || this.drawT>0 ||
          this.reloading || this.grenadeCooking || this.charging ||
          g.time < (this.nextFire||0)) {
        return runOriginal();
      }
      const p=g.player;
      const origin={x:p.x,y:p.y+Number(p.eyeH??1.55)-(p._stepLift||0),z:p.z};
      const target=rageTarget(g,origin);
      if (!target) return runOriginal();
      rageLastTarget=target;
      if (cfg.rageAutoStop && p.onGround && !p.onLadder) {
        p.vx=0; p.vz=0;
      }
      return runForcedFire();
    };
    Object.defineProperty(wrapper,'__amnesisRageBot',{value:true});
    w.update=wrapper;
    rageHookedWeapons=w;
  }

  setInterval(ensureRageFireHook,100);


  function aimTarget(g, sx, sy, sz) {
    const canvas = document.querySelector('#game canvas') || g.renderer?.domElement;
    const camera = (cfg.thirdPerson && g.__amnesisThirdCamera) || g.camera;

    if (!canvas || !camera ||
        !(g.botMgr?.bots || typeof g.allEntities === 'function')) return null;

    const rect = canvas.getBoundingClientRect();
    if (!rect.width || !rect.height) return null;

    camera.updateMatrixWorld?.(true);

    const view = camera.matrixWorldInverse?.elements;
    const projection = camera.projectionMatrix?.elements;

    if (!view || !projection) return null;

    const centerX = rect.width / 2;
    const centerY = rect.height / 2;
    const radius = Number(cfg.silentFov) || 120;

    let best = null;
    let bestDistance = radius;

    const bots = characterEntities(g);

    for (const bot of bots) {
      if (!bot?.alive || bot.team === g.player?.team) continue;
      if (bot.spawnProt > 0) continue;

      const agent = characterAgent(bot);
      const head = agent?.bones?.head_0 || agent?.bones?.head;
      if (!head) continue;

      head.updateWorldMatrix?.(true, false);
      const m = head.matrixWorld?.elements;
      if (!m) continue;

      const projected = point(head, view, projection, rect.width, rect.height);
      if (!projected) continue;

      const distance = Math.hypot(
        projected[0] - centerX,
        projected[1] - centerY
      );

      if (distance >= bestDistance) continue;

      const dx = m[12] - sx;
      const dy = m[13] - sy;
      const dz = m[14] - sz;
      const length = Math.hypot(dx, dy, dz);

      if (length < 0.01) continue;

      best = [dx / length, dy / length, dz / length];
      bestDistance = distance;
    }

    return best;
  }

  function triggerTarget(g, origin) {
    const canvas = document.querySelector('#game canvas') || g.renderer?.domElement;
    const camera = g.camera;
    if (!canvas || !camera) return false;
    const rect = canvas.getBoundingClientRect();
    if (!rect.width || !rect.height) return false;
    camera.updateMatrixWorld?.(true);
    const view = camera.matrixWorldInverse?.elements;
    const projection = camera.projectionMatrix?.elements;
    if (!view || !projection) return false;
    const radius = Math.max(2, Math.min(40, Number(cfg.triggerRadius) || 8));
    const cx = rect.width / 2, cy = rect.height / 2;
    for (const enemy of characterEntities(g)) {
      if (!enemy?.alive || enemy.team === g.player?.team || enemy.spawnProt > 0) continue;
      const agent = characterAgent(enemy);
      const head = agent?.bones?.head_0 || agent?.bones?.head;
      if (!head) continue;
      head.updateWorldMatrix?.(true, false);
      const m = head.matrixWorld?.elements;
      const projected = point(head, view, projection, rect.width, rect.height);
      if (!m || !projected || Math.hypot(projected[0]-cx, projected[1]-cy) > radius) continue;
      let visible = false;
      try {
        visible = g.physics?.lineClear?.(origin.x,origin.y,origin.z,m[12],m[13],m[14]) === true;
        if (visible && !cfg.noSmoke && typeof g.grenadeMgr?.lineBlockedBySmoke === 'function') {
          visible = !g.grenadeMgr.lineBlockedBySmoke(
            origin.x,origin.y,origin.z,m[12],m[13],m[14]
          );
        }
      } catch { visible = false; }
      if (visible) return true;
    }
    return false;
  }

  function ensureFireHook() {
    const g = window.game;
    if (!g || typeof g.fireBullet !== 'function') return;
    if (g === hookedGame && g.fireBullet.__clutcherSilentAim) return;

    const original = g.fireBullet;

    const wrapper = function(shooter, x, y, z, dx, dy, dz, weapon) {
      let bypassWorld = false, rageResolved = false;
      if (shooter === this.player && this.state === 'playing') {
        if (cfg.rageBot && rageCombatActive(this,true)) {
          try {
            const resolved = rageTarget(this, {x,y,z});
            if (resolved) {
              [dx,dy,dz]=[resolved.dir.x,resolved.dir.y,resolved.dir.z];
              bypassWorld = !!cfg.rageFullMapWallbang;
              rageResolved = true;
            }
          } catch (error) {
            console.warn('[Rage Bot] targeting:', error);
          }
        } else if (cfg.silentAim) {
          try {
            const target = aimTarget(this, x, y, z);
            if (target) [dx, dy, dz] = target;
          } catch (error) {
            console.warn('[Silent Aim] targeting:', error);
          }
        }
      }
      if (rageResolved && cfg.rageDamageBoost && weapon) {
        const percent=Math.max(10,Math.min(500,Number(cfg.rageDamagePercent)||150));
        weapon={...weapon,dmg:(Number(weapon.dmg)||0)*percent/100};
      }
      const fire = () => original.call(this, shooter, x, y, z, dx, dy, dz, weapon);
      const physics = this.physics;
      if (!bypassWorld || typeof physics?.castIntervals !== 'function') return fire();
      const nativeCastIntervals = physics.castIntervals;
      const ownDescriptor=Object.getOwnPropertyDescriptor(physics,'castIntervals');
      const bulletGame=this;
      const bypassCastIntervals = function(...args) {
        if (this===physics&&args[7]===2&&Array.isArray(args[8])&&
            args[8]===bulletGame._bulletPath?._w) {
          args[8].length = 0;
          return args[8];
        }
        return nativeCastIntervals.apply(this, args);
      };
      try { physics.castIntervals = bypassCastIntervals; }
      catch { return fire(); }
      if (physics.castIntervals !== bypassCastIntervals) return fire();
      try {
        return fire();
      } finally {
        if (ownDescriptor) Object.defineProperty(physics,'castIntervals',ownDescriptor);
        else delete physics.castIntervals;
      }
    };

    Object.defineProperty(wrapper, '__clutcherSilentAim', { value: true });

    g.fireBullet = wrapper;
    hookedGame = g;
  }

  setInterval(ensureFireHook, 100);

  const smokeHookedManagers = new WeakSet();
  const bloodHookedEffects = new WeakSet();

  function ensureGameplayFeatureHooks() {
    const g = window.game;
    if (!g) return;
    if (typeof g.playerShoot === 'function' && !g.playerShoot.__amnesisNoSpread) {
      const original = g.playerShoot;
      const wrapper = function(weapon, spreadX, spreadY, ...rest) {
        if (cfg.noSpread) { spreadX = 0; spreadY = 0; }
        return original.call(this, weapon, spreadX, spreadY, ...rest);
      };
      Object.defineProperty(wrapper, '__amnesisNoSpread', {value:true});
      g.playerShoot = wrapper;
    }
    if (typeof g.flashPlayer === 'function' && !g.flashPlayer.__amnesisNoFlash) {
      const original = g.flashPlayer;
      const wrapper = function(...args) {
        if (cfg.removeFlash) return;
        return original.apply(this, args);
      };
      Object.defineProperty(wrapper, '__amnesisNoFlash', {value:true});
      g.flashPlayer = wrapper;
    }
    const smoke = g.grenadeMgr;
    if (smoke && !smokeHookedManagers.has(smoke)) {
      for (const [name, disabledValue] of [
        ['bloomSmoke', undefined], ['lineBlockedBySmoke', false],
        ['smokeGrey', 0], ['smokeCover', 0]
      ]) {
        if (typeof smoke[name] !== 'function') continue;
        const original = smoke[name];
        smoke[name] = function(...args) {
          if (cfg.noSmoke) return disabledValue;
          return original.apply(this, args);
        };
      }
      smokeHookedManagers.add(smoke);
    }
    const effects=g.effects;
    if (effects&&!bloodHookedEffects.has(effects)) {
      for (const name of ['blood','bloodSplat']) {
        if (typeof effects[name]!=='function') continue;
        const original=effects[name];
        effects[name]=function(...args) {
          if (cfg.noBlood) return;
          return original.apply(this,args);
        };
      }
      bloodHookedEffects.add(effects);
    }
  }

  setInterval(ensureGameplayFeatureHooks, 100);

  // =====================================================
  // AUTO BHOP
  // =====================================================

  // Tracks the real (trusted) Space key, then re-sends a fresh Space keydown
  // every frame while it's held so a jump registers the moment you land.
  let spaceHeld = false;

  window.addEventListener('keydown', e => {
    if (e.isTrusted && e.code === 'Space') spaceHeld = true;
  }, true);

  window.addEventListener('keyup', e => {
    if (e.isTrusted && e.code === 'Space') spaceHeld = false;
  }, true);

  window.addEventListener('blur', () => { spaceHeld = false; });

  function bhopFrame() {
    requestAnimationFrame(bhopFrame);

    if (!(cfg.autoBhop || cfg.superBhop) || !spaceHeld || uiOpen) return;
    if (window.game?.state !== 'playing') return;

    const target = document.querySelector('#game canvas') || document.body;
    if (!target) return;

    try {
      target.dispatchEvent(new KeyboardEvent('keydown', {
        key: ' ', code: 'Space', keyCode: 32, which: 32,
        repeat: false, bubbles: true, cancelable: true
      }));
    } catch (error) {
      console.warn('[Auto Bhop]', error);
    }
  }

  requestAnimationFrame(bhopFrame);

  function applySuperBhopBoost(player, g) {
    if (!cfg.superBhop || !spaceHeld || uiOpen || g?.state !== 'playing' ||
        !player?.alive || player.onGround || player.onLadder) return;
    const baseSpeed = Number(g.weapons?.maxSpeed?.());
    if (!(baseSpeed > 0)) return;
    const percent = Math.max(100, Math.min(500,
      Number(cfg.superBhopSpeedPercent) || 250));
    const targetSpeed = baseSpeed * percent / 100;
    const currentSpeed = Math.hypot(Number(player.vx) || 0, Number(player.vz) || 0);
    if (currentSpeed >= targetSpeed) return;
    if (currentSpeed > 0.01) {
      const scale = targetSpeed / currentSpeed;
      player.vx *= scale;
      player.vz *= scale;
      return;
    }
    player.vx = -Math.sin(Number(player.yaw) || 0) * targetSpeed;
    player.vz = -Math.cos(Number(player.yaw) || 0) * targetSpeed;
  }

  // =====================================================
  // THIRD PERSON
  // =====================================================
  // Clutcher has two independent renders: the world camera and a
  // first-person weapon scene. Shift only the WORLD render so mouse,
  // movement, bullet origins, audio and the viewmodel rig stay native.
  // The game's existing animated lobby character provides the body.

  const tp = {
    game: null, agent: null, weapon: null, paint: null, glove: null, lastAt: 0,
    visualCamera: null, inRender: false, pendingActions: [], deathUntil: 0
  };

  const tpPostHooks = new WeakSet();
  const tpRendererHooks = new WeakSet();
  const tpWeaponHooks = new WeakSet();
  const tpGameHooks = new WeakSet();
  const tpViewmodelHooks = new WeakSet();

  function thirdPersonActive(g) {
    return !!(
      cfg.thirdPerson && g && g.state === 'playing' &&
      g.player && (g.player.alive || performance.now() < tp.deathUntil) &&
      g.camera && g.scene &&
      !g._topCam && !g._skinView && !g._buyPreview &&
      !g._caseView && !g.scoped
    );
  }

  function returnThirdPersonBody(g) {
    const agent = tp.agent;
    if (agent?.root) {
      // Only move the character back if the game is still using it
      // as its menu/buy preview. It may have loaded a replacement.
      if (g?.previewAgent === agent && g.previewScene) {
        if (agent.root.parent !== g.previewScene) {
          g.previewScene.add(agent.root);
        }
        agent.root.position.set(0.35, 0, 0);
      } else if (agent.root.parent === g?.scene) {
        g.scene.remove(agent.root);
      }
    }
    tp.agent = null;
    tp.weapon = null;
    tp.paint = null;
    tp.glove = null;
    tp.lastAt = 0;
    tp.pendingActions.length = 0;
    tp.deathUntil = 0;
    if (g) g.__amnesisThirdCamera = null;
  }

  function thirdPersonWeaponType(id) {
    if (isKnife(id)) return 'knife';
    if (/^(?:glock|usp|p2000|p250|fiveseven|tec9|cz75|deagle|revolver|dual|elite)/i.test(id)) {
      return 'pistol';
    }
    if (/^(?:mac10|mp5|mp7|mp9|ump|p90|bizon)/i.test(id)) return 'smg';
    if (/^(?:nova|mag7|xm1014|sawedoff)/i.test(id)) return 'shotgun';
    if (/^(?:m249|negev)/i.test(id)) return 'machinegun';
    if (/^(?:he|hegrenade|flash|flashbang|smoke|smokegrenade|molotov|incend|incgrenade|decoy)$/i.test(id)) return 'grenade';
    if (id === 'c4') return 'c4';
    return 'rifle';
  }

  // Resolve the exact third-person weapon ID, including the team's knife
  // (knife_ct / knife_t) and any equipped knife model. The game exposes
  // this through _povLoadout; using 'knife' directly can fail to load a
  // world model and prevents the correct action clips from being selected.
  function thirdPersonHeldWeapon(g) {
    const current = g.weapons?.current;
    if (!current) return null;
    let loadout,id;
    try { loadout = g._povLoadout?.(g.player); id=loadout?.wid; } catch {}
    if (!id) {
      id = current === 'knife'
        ? (g.player?.team === 'T' ? 'knife_t' : 'knife_ct')
        : current;
    }
    return {
      id,
      type: g.weapons?.def?.()?.class || thirdPersonWeaponType(current),
      paintOpts:loadout?.paintOpts||{}
    };
  }

  // The world player rig has its own animation controller (unlike the
  // first-person viewmodel). Apply actions only after setWeapon has
  // finished loading its world model, not while it still has the lobby gun.
  function thirdPersonDispatchAction(g, action) {
    const agent = tp.agent;
    if (!agent?.loco || agent._weaponId !== action.weaponId ||
        !thirdPersonActive(g)) return false;
    try {
      if (action.kind === 'knife') {
        agent.knifeAttack?.(action.heavy ? 'stab' : 'swing');
      } else if (action.kind === 'reload') {
        const elapsed = (performance.now() - action.at) / 1000;
        const remaining = Math.max(0.3, action.duration - elapsed);
        agent.reload?.(action.weaponId, remaining, {
          empty: action.empty, crouched: action.crouched
        });
      } else if (action.kind === 'shoot') {
        agent.shoot?.(action.weaponId, { crouched: action.crouched });
      } else if (action.kind === 'pin') {
        agent.pullPin?.(action.weaponId);
      } else if (action.kind === 'throw') {
        agent.throwGrenade?.({
          under: action.under, crouched: action.crouched
        });
      }
      return true;
    } catch (error) {
      console.warn('[Third Person] action:', error);
      return true;
    }
  }

  function thirdPersonAction(g, kind, details = {}) {
    if (!thirdPersonActive(g)) return;
    const weapon = thirdPersonHeldWeapon(g);
    if (!weapon) return;
    const event = {
      kind, weaponId: weapon.id, at: performance.now(), ...details
    };
    if (!thirdPersonDispatchAction(g, event)) {
      // A new weapon model loads asynchronously. Permit a brief delay for
      // the action to reach the right rig without replaying old swings.
      tp.pendingActions.push(event);
      if (tp.pendingActions.length > 12) tp.pendingActions.shift();
    }
  }

  function thirdPersonFlushActions(g, currentId) {
    const now = performance.now();
    tp.pendingActions = tp.pendingActions.filter(action => {
      const ttl = action.kind === 'reload' ? 1200 :
        action.kind === 'pin' ? 900 : 450;
      if (action.weaponId !== currentId || now - action.at > ttl) {
        return false;
      }
      return !thirdPersonDispatchAction(g, action);
    });
  }

  function ensureThirdPersonActionHooks(g) {
    const weapons = g?.weapons;
    if (!weapons || tpWeaponHooks.has(weapons)) return;

    // Native gameplay remains unchanged; these wrappers only send visual
    // animation events to the separate third-person world character.
    if (typeof weapons.knifeAttack === 'function') {
      const original = weapons.knifeAttack;
      weapons.knifeAttack = function(heavy, ...args) {
        const result = original.call(this, heavy, ...args);
        if (this.current === 'knife') {
          thirdPersonAction(g, 'knife', { heavy: !!heavy });
        }
        return result;
      };
    }

    if (typeof weapons.startReload === 'function') {
      const original = weapons.startReload;
      weapons.startReload = function(...args) {
        const definition = this.def?.();
        const state = this.state?.();
        const empty = Number(state?.ammo) <= 0;
        const amount = Math.max(0, Math.min(
          (definition?.mag || 0) - (state?.ammo || 0),
          state?.reserve || 0
        ));
        const duration = definition?.shellReload
          ? Math.max(definition.reload || 0, amount * (definition.reload || 0))
          : (definition?.reload || 1);
        const result = original.apply(this, args);
        if (this.reloading) {
          thirdPersonAction(g, 'reload', {
            duration, empty,
            crouched: !!g.player?.crouching
          });
        }
        return result;
      };
    }

    if (typeof weapons.fire === 'function') {
      const original = weapons.fire;
      weapons.fire = function(...args) {
        const current = this.current;
        const result = original.apply(this, args);
        if (this.current === current) {
          thirdPersonAction(g, 'shoot', {
            crouched: !!g.player?.crouching
          });
        }
        return result;
      };
    }

    // Weapon release produces the third-person overhand / underhand
    // grenade clip. Capture the throw direction before the game clears it.
    if (typeof weapons.throwGrenade === 'function') {
      const original = weapons.throwGrenade;
      weapons.throwGrenade = function(...args) {
        const id = this.current;
        const under = !!this.grenadeUnder;
        const wasCooking = !!this.grenadeCooking;
        const result = original.apply(this, args);
        if (wasCooking && thirdPersonWeaponType(id) === 'grenade') {
          thirdPersonAction(g, 'throw', {
            under, crouched: !!g.player?.crouching
          });
        }
        return result;
      };
    }

    tpWeaponHooks.add(weapons);
  }

  function ensureThirdPersonExtraHooks(g) {
    // The weapon system calls viewmodel.pullPin once when cooking starts.
    // Mirror it to the separate third-person rig, keeping native controls.
    const vm = g?.viewmodel;
    if (vm && !tpViewmodelHooks.has(vm)) {
      if (typeof vm.pullPin === 'function') {
        const original = vm.pullPin;
        vm.pullPin = function(...args) {
          const result = original.apply(this, args);
          if (g.weapons?.grenadeCooking &&
              thirdPersonWeaponType(g.weapons?.current) === 'grenade') {
            thirdPersonAction(g, 'pin');
          }
          return result;
        };
      }
      tpViewmodelHooks.add(vm);
    }
    if (tpGameHooks.has(g)) return;

    // The game only applies these reactions to bot world rigs. Apply the
    // same hit-group and direction to the player's third-person rig.
    function reactToHit(victim, attacker, headshot, weapon, hitgroup, hitSide) {
      if (victim !== g.player || !victim?.alive ||
          !thirdPersonActive(g) || !tp.agent) return;
      let zone = weapon && /molotov|incend/i.test(weapon.id || weapon.name || '')
        ? 'molotov'
        : headshot || /head|neck/i.test(String(hitgroup || ''))
          ? 'head'
          : /stomach|leg|arm/i.test(String(hitgroup || ''))
            ? String(hitgroup).toLowerCase() : 'chest';
      let side = 'f';
      if (zone === 'leg' || zone === 'arm') {
        side = hitSide === 4 || hitSide === 6 ? 'l' : 'r';
      } else if (attacker && attacker !== victim &&
                 Number.isFinite(attacker.x) && Number.isFinite(attacker.z)) {
        const dx = attacker.x - victim.x;
        const dz = attacker.z - victim.z;
        const sin = Math.sin(victim.yaw || 0);
        const cos = Math.cos(victim.yaw || 0);
        const front = dx * -sin + dz * -cos;
        const lateral = dx * cos - dz * sin;
        side = Math.abs(lateral) > Math.abs(front)
          ? (lateral > 0 ? 'r' : 'l') : (front < 0 ? 'b' : 'f');
      }
      try { tp.agent.flinch?.(zone, side); }
      catch (error) { console.warn('[Third Person] flinch:', error); }
    }

    if (typeof g.dealDamage === 'function') {
      const original = g.dealDamage;
      g.dealDamage = function(victim, amount, attacker, headshot, weapon,
                               hitgroup, hitSide, ...rest) {
        const before = victim?.health;
        const dealtBefore=Number(this.player?._dmgDealt)||0;
        if (cfg.lethalShots && attacker === this.player && victim !== attacker &&
            victim?.alive && victim.team !== attacker?.team &&
            firearmClasses.has(weapon?.class)) {
          const health = Math.max(0, Number(victim.health) || 0);
          const armor = Math.max(0, Number(victim.armor) || 0);
          amount = Math.max(Number(amount) || 0, health + armor * 2 + 1);
        }
        const result = original.call(this, victim, amount, attacker, headshot,
          weapon, hitgroup, hitSide, ...rest);
        if (attacker===this.player) {
          sessionStats.damage+=Math.max(0,(Number(this.player?._dmgDealt)||0)-dealtBefore);
        }
        if (Number.isFinite(before) && victim?.health < before) {
          reactToHit(victim, attacker, headshot, weapon, hitgroup, hitSide);
        }
        return result;
      };
    }
    if (typeof g.damagePlayer === 'function') {
      const original = g.damagePlayer;
      g.damagePlayer = function(...args) {
        const before = this.player?.health;
        const result = original.apply(this, args);
        if (Number.isFinite(before) && this.player?.health < before) {
          reactToHit(this.player, args[1], false, null, 'chest', 0);
        }
        return result;
      };
    }

    // Keep the third-person body in view briefly on death so the game's
    // real ragdoll-style death clip can play before the usual spectator view.
    if (typeof g.killEntity === 'function') {
      const original = g.killEntity;
      g.killEntity = function(victim, attacker, weapon, headshot, hitgroup, ...rest) {
        const killsBefore=Number(this.player?.kills)||0;
        if (victim === this.player && victim?.alive && cfg.thirdPerson &&
            tp.agent?.loco && this.state === 'playing') {
          try {
            const region = headshot ? 'head' :
              hitgroup === 'stomach' || hitgroup === 'leg' ? hitgroup : 'chest';
            const duration = Number(tp.agent.die?.(
              region, !!victim.crouching, !!headshot
            ));
            tp.deathUntil = performance.now() +
              Math.min(2800, Math.max(950, (duration || 1.8) * 1000));
          } catch (error) {
            console.warn('[Third Person] death:', error);
          }
        }
        const wasAlive = !!victim?.alive;
        const result = original.call(this, victim, attacker, weapon, headshot,
          hitgroup, ...rest);
        sessionStats.kills+=Math.max(0,(Number(this.player?.kills)||0)-killsBefore);
        // Fire only for the local player's confirmed enemy kill, not every hit.
        const local = this.player;
        const creditedToLocal = attacker === local ||
          (attacker != null && local != null &&
           (attacker === local.id || attacker?.id === local.id && local.id != null));
        if (wasAlive && victim !== local && victim?.alive === false &&
            creditedToLocal && (victim.team == null || victim.team !== local?.team)) {
          playHitsound();
          if (cfg.skinRandomKnife||cfg.skinRandomGloves) rerollRuntimeSkins();
          if (cfg.autoInspectKill) pendingInspect={
            weapon:this.weapons?.current,
            expires:performance.now()+1500
          };
        }
        return result;
      };
    }
    tpGameHooks.add(g);
  }

  // =====================================================
  // RAGE: ANTI AIM (VISUAL THIRD-PERSON MODEL)
  // =====================================================
  // Only the visible character rig changes. Changing player.yaw/pitch
  // would redirect shots, input, and the actual first-person camera.
  // Recompute from the physical eye: visible crosshair target first,
  // then nearest visible enemy, then nearest hidden enemy.
  const aaNumber = (value, min, max, fallback) => {
    const number = Number(value);
    return Number.isFinite(number) ? Math.max(min, Math.min(max, number)) : fallback;
  };

  function antiAimOffsets(settings, now) {
    const radians = Math.PI / 180;
    const interval = aaNumber(settings.aaInterval, 40, 1000, 120);
    const step = Math.floor(Math.max(0, now) / interval);
    const side = step % 2 ? 1 : -1;
    const range = aaNumber(settings.aaJitterRange, 0, 180, 60);
    let jitter = 0;
    if (settings.aaJitter === 'center') jitter = side * range / 2;
    if (settings.aaJitter === 'offset') jitter = step % 2 ? range : 0;
    if (settings.aaJitter === 'three') jitter = [-range / 2, 0, range / 2][step % 3];
    // Deterministic per interval: drawing more frames never speeds up jitter.
    if (settings.aaJitter === 'random') {
      const random = Math.sin((step + 1) * 12.9898) * 43758.5453;
      jitter = ((random - Math.floor(random)) - .5) * range;
    }
    const amount = aaNumber(settings.aaDesyncAmount, 0, 60, 35);
    let body = 0;
    if (settings.aaDesync === 'static') body = amount;
    if (settings.aaDesync === 'alternate') body = side * amount;
    if (settings.aaDesync === 'sway') body = Math.sin(now / interval * Math.PI) * amount;
    if (settings.aaInvert) body = -body;
    const spin = settings.aaYawMode === 'spin' ?
      now / 1000 * aaNumber(settings.aaSpinSpeed, -360, 360, 90) : 0;
    return (aaNumber(settings.aaYawOffset, -180, 180, 0) + jitter + body + spin) * radians;
  }

  function antiAimModelPose(g) {
    const p = g?.player;
    if (!cfg.antiAim || !p?.alive) return null;

    // Use the REAL player eye/head position and aim angles, never the
    // offset third-person chase camera or an Anti-Aim-rotated skeleton.
    const hx = Number(p.x);
    const hy = Number(p.y) + Number(p.eyeH ?? 1.6256);
    const hz = Number(p.z);
    const yaw = Number(p.yaw);
    const pitch = Number(p.pitch);
    if (![hx, hy, hz, yaw, pitch].every(Number.isFinite)) return null;

    const enemies = cfg.aaBase === 'view' ? [] : characterEntities(g);

    // Native forward direction, matching Clutcher's player/camera code.
    const cosPitch = Math.cos(pitch);
    const forwardX = -Math.sin(yaw) * cosPitch;
    const forwardY = Math.sin(pitch);
    const forwardZ = -Math.cos(yaw) * cosPitch;
    const cosCrosshairCone = Math.cos(12 * Math.PI / 180);
    const physics = g.physics;
    const smoke = g.grenadeMgr;

    let bestCrosshair = null;
    let bestCrosshairDot = -Infinity;
    let bestCrosshairDistSq = Infinity;
    let nearestVisible = null;
    let nearestVisibleDistSq = Infinity;
    let nearestOverall = null;
    let nearestOverallDistSq = Infinity;

    for (const enemy of enemies) {
      if (!enemy?.alive || enemy === p || enemy.team === p.team) continue;

      const ex = Number(enemy.x);
      const ey = Number(enemy.y);
      const ez = Number(enemy.z);
      const enemyEye = Number(enemy.eyeH ?? 1.6256);
      const enemyHeight = Number(enemy.height ?? 1.8);
      if (![ex, ey, ez, enemyEye, enemyHeight].every(Number.isFinite)) continue;

      const dx = ex - hx;
      const dy = ey + enemyEye - hy;
      const dz = ez - hz;
      const distSq = dx * dx + dy * dy + dz * dz;
      if (!(distSq > 0.0001)) continue;

      const result = { dx, dz };
      if (distSq < nearestOverallDistSq) {
        nearestOverallDistSq = distSq;
        nearestOverall = result;
      }

      // Sample the physical head and upper/lower torso. This allows
      // partial exposure (e.g., peeking a wall) to count as visible,
      // while targets fully behind world geometry do not.
      const samples = [
        ey + enemyEye,
        ey + enemyHeight * 0.70,
        ey + enemyHeight * 0.43
      ];
      let visible = false;
      let bestDot = -Infinity;

      for (const sy of samples) {
        let clear = false;
        try {
          clear = typeof physics?.lineClear === 'function' &&
            physics.lineClear(hx, hy, hz, ex, sy, ez);
          if (clear && typeof smoke?.lineBlockedBySmoke === 'function') {
            clear = !smoke.lineBlockedBySmoke(hx, hy, hz, ex, sy, ez);
          }
        } catch { clear = false; }
        if (!clear) continue;
        visible = true;

        const vx = ex - hx;
        const vy = sy - hy;
        const vz = ez - hz;
        const length = Math.hypot(vx, vy, vz);
        if (length <= 0.0001) continue;
        const dot = (forwardX * vx + forwardY * vy + forwardZ * vz) / length;
        if (dot > bestDot) bestDot = dot;
      }

      if (!visible) continue;

      if (distSq < nearestVisibleDistSq) {
        nearestVisibleDistSq = distSq;
        nearestVisible = result;
      }

      // Crosshair priority ONLY applies to visible targets near the
      // actual aim direction. Within the cone, prefer the closest
      // alignment to the crosshair, then the shorter distance.
      if (bestDot >= cosCrosshairCone &&
          (bestDot > bestCrosshairDot + 0.00001 ||
           (Math.abs(bestDot - bestCrosshairDot) <= 0.00001 &&
            distSq < bestCrosshairDistSq))) {
        bestCrosshair = result;
        bestCrosshairDot = bestDot;
        bestCrosshairDistSq = distSq;
      }
    }

    // PRIORITY: visible crosshair target -> nearest visible enemy ->
    // nearest enemy (only when every enemy is hidden/occluded).
    // Thus a wall-hidden crosshair target cannot override a guy who
    // walks up behind the player in clear sight.
    const selected = bestCrosshair || nearestVisible || nearestOverall;
    const base = selected ? Math.atan2(-selected.dx, -selected.dz) : yaw;
    const backward = cfg.aaYawMode === 'backward' ? Math.PI : 0;
    const angle = base + backward + antiAimOffsets(cfg, performance.now());
    const pitchModes = {down: -Math.PI / 2 + .07, up: Math.PI / 2 - .07, level: 0, view: pitch};
    return {
      yaw: Math.atan2(Math.sin(angle), Math.cos(angle)),
      pitch: pitchModes[cfg.aaPitch] ?? pitchModes.down
    };
  }

  function renderThirdPersonBody(g) {
    const p = g.player;
    // The lobby preview defaults to CT; load the player's actual side.
    if (p.team && g.previewCharTeam !== p.team) {
      g.setPreviewTeam?.(p.team);
    }
    const agent = g.previewAgent;
    if (tp.game !== g) {
      returnThirdPersonBody(tp.game);
      tp.game = g;
    }
    if (!agent?.root) {
      returnThirdPersonBody(g);
      return;
    }
    if (tp.agent && tp.agent !== agent) {
      if (tp.agent.root?.parent === g.scene) g.scene.remove(tp.agent.root);
      tp.weapon = null;
      tp.paint = null;
      tp.glove = null;
      tp.pendingActions.length = 0;
    }
    tp.agent = agent;
    if (agent.root.parent !== g.scene) g.scene.add(agent.root);

    let floorY = p.y;
    try {
      const ground = g.physics?.groundHeight?.(p.x, p.y + 1.2, p.z);
      if (p.onGround && Number.isFinite(ground) && ground > -20) {
        floorY = g.map?.visualGround?.(p.x, p.z, ground) ?? ground;
      }
    } catch {}

    // The rendered body looks away from the dynamically selected enemy.
    // Player yaw/pitch, hitboxes, camera, and shooting remain native.
    const antiPose = antiAimModelPose(g);
    agent.setTransform?.(p.x, floorY, p.z, antiPose?.yaw ?? p.yaw);
    agent.root.visible = true;

    // A respawn restores the normal locomotion state after a death clip.
    if (p.alive && agent.dead) {
      agent.revive?.();
      tp.deathUntil = 0;
    }

    const now = performance.now();
    const dt = tp.lastAt ? Math.min(0.05, Math.max(0, (now - tp.lastAt) / 1000)) : 0;
    tp.lastAt = now;
    // These are the same input flags Clutcher passes to bot animation rigs.
    // Loco blends idle, walk/run, strafe, crouch, jumping, and landing.
    const mode = g.modeCtl;
    const planting = !!(mode &&
      (mode._planting === p ||
       (mode.carrier === p && mode.plantProgress > 0)));
    const defusing = !!(mode && mode.defuser === p &&
      mode.defuseProgress > 0);
    const flash = Math.max(0, Math.min(1,
      ((p.flashUntil || 0) - (g.time || 0)) / 1.2
    ));
    agent.update?.(dt, {
      vx: p.vx || 0, vz: p.vz || 0,
      crouch: p.crouching || p.crouchT > 0 ? 1 : 0,
      pitch: antiPose?.pitch ?? (p.pitch || 0), airborne: !p.onGround,
      planting, defusing, flashed: flash
    });

    // Freeze the weapon while the death clip is playing.
    if (!p.alive) return;
    const heldWeapon=thirdPersonHeldWeapon(g);
    const weapon = planting
      ? { id: 'c4', type: 'c4', paintOpts:heldWeapon?.paintOpts||{} }
      : heldWeapon;
    const glove=weapon?.paintOpts?.glove;
    const gloveKey=glove?[glove.model,glove.kit||'',glove.item?.seed||0,
      glove.item?.wear||0].join('|'):'default';
    if (tp.glove!==gloveKey) {
      tp.glove=gloveKey;
      agent._gloveReq=null;
      agent.setGloves?.(glove?.model||null,
        glove?.kit?{kit:glove.kit,item:glove.item}:null,{size:512});
    }
    const paint=weapon?.paintOpts?.kit?
      {kit:weapon.paintOpts.kit,item:weapon.paintOpts.item}:null;
    const paintKey=weapon?[weapon.id,paint?.kit||'',paint?.item?.uid||
      paint?.item?.seed||'',paint?.item?.wear??''].join('|'):'';
    if (weapon && (tp.weapon !== weapon.id ||
        tp.paint !== paintKey ||
        agent._carryId !== weapon.id)) {
      tp.weapon = weapon.id;
      tp.paint = paintKey;
      try {
        agent._weaponReq=null;
        const load = agent.setWeapon?.(weapon.id, weapon.type, paint);
        load?.then?.(ready => {
          if (ready && tp.agent === agent && tp.weapon === weapon.id &&
              thirdPersonActive(g)) {
            // Game bots use this same draw action when swapping weapons.
            agent.draw?.(weapon.id, { crouched: !!p.crouching });
            thirdPersonFlushActions(g, weapon.id);
          }
        })?.catch?.(err => console.warn('[Third Person] weapon:', err));
      } catch (err) {
        console.warn('[Third Person] weapon:', err);
      }
    }
    if (weapon) thirdPersonFlushActions(g, weapon.id);
  }

  function thirdPersonWorldRender(g, render) {
    if (tp.inRender) return render();
    const weaponScene = g.viewmodel?.scene;
    if (!thirdPersonActive(g)) {
      if (tp.agent) returnThirdPersonBody(tp.game || g);
      g.__amnesisThirdCamera = null;
      if (!cfg.hideViewmodel || !weaponScene) return render();
      const visible = weaponScene.visible;
      try {
        weaponScene.visible = false;
        return render();
      } finally {
        weaponScene.visible = visible;
      }
    }

    const camera = g.camera;
    const originalPosition = camera.position.clone();
    const originalQuaternion = camera.quaternion.clone();
    const originalVisibility = weaponScene?.visible;
    tp.inRender = true;

    try {
      renderThirdPersonBody(g);

      // True centered chase camera: pivot around the middle of the player.
      // Both yaw and pitch orbit this pivot, instead of leaving the body
      // off to one side like an over-the-shoulder aiming camera.
      camera.updateMatrixWorld(true);
      const m = camera.matrixWorld.elements;
      const dirLength = Math.hypot(m[8], m[9], m[10]) || 1;
      // Backward is the positive camera-Z axis in Three.js.
      const backX = m[8] / dirLength;
      const backY = m[9] / dirLength;
      const backZ = m[10] / dirLength;
      const player = g.player;
      const pivotX = player.x;
      // Keep the pivot at neck/head height, just below the player's eyes.
      // This follows crouching naturally without changing the camera offset.
      const eyeHeight = Number.isFinite(player.eyeH) ? player.eyeH : 1.55;
      const pivotY = (tp.agent?.root?.position?.y ?? player.y) +
        Math.max(0.72, eyeHeight - 0.10);
      const pivotZ = player.z;
      // Live-adjustable chase distance; never exceeds the slider's range.
      const rawDistance = Number(cfg.thirdDistance);
      const desiredDistance = Number.isFinite(rawDistance)
        ? Math.max(0.02, Math.min(5, rawDistance))
        : 3.30;

      // Check from the character's center outwards along the entire camera
      // orbit path. This also works when looking sharply up or down.
      let distance = desiredDistance;
      try {
        const hit = g.physics?.raycast?.(
          pivotX, pivotY, pivotZ, backX, backY, backZ,
          desiredDistance
        );
        if (Number.isFinite(hit?.t) && hit.t >= 0) {
          const clearance = Math.min(0.2, desiredDistance * 0.25);
          distance = Math.max(0.02,
            Math.min(desiredDistance, hit.t - clearance));
        }
      } catch {}

      camera.position.set(
        pivotX + backX * distance,
        pivotY + backY * distance,
        pivotZ + backZ * distance
      );
      camera.lookAt(pivotX, pivotY, pivotZ);
      camera.updateMatrixWorld(true);

      // Store the actual third-person projection so ESP/FOV overlays
      // stay aligned with what appears on screen.
      if (!tp.visualCamera) tp.visualCamera = camera.clone();
      else tp.visualCamera.copy(camera);
      tp.visualCamera.updateMatrixWorld(true);
      g.__amnesisThirdCamera = tp.visualCamera;

      // Suppress just the floating first-person gun/arms.
      if (weaponScene) weaponScene.visible = false;
      return render();
    } catch (err) {
      console.warn('[Third Person] render:', err);
      return render();
    } finally {
      camera.position.copy(originalPosition);
      camera.quaternion.copy(originalQuaternion);
      camera.updateMatrixWorld(true);
      if (weaponScene) weaponScene.visible = originalVisibility;
      tp.inRender = false;
    }
  }

  function ensureThirdPersonHooks() {
    const g = window.game;
    if (!g) return;

    ensureThirdPersonActionHooks(g);
    ensureThirdPersonExtraHooks(g);

    if (g.postfx && typeof g.postfx.render === 'function' &&
        !tpPostHooks.has(g.postfx)) {
      const post = g.postfx;
      const original = post.render;
      post.render = function(...args) {
        return thirdPersonWorldRender(g, () => original.apply(this, args));
      };
      tpPostHooks.add(post);
    }

    // Fallback for a build where post-processing is disabled.
    if (g.renderer && typeof g.renderer.render === 'function' &&
        !tpRendererHooks.has(g.renderer)) {
      const renderer = g.renderer;
      const original = renderer.render;
      renderer.render = function(scene, camera, ...args) {
        if (!tp.inRender && cfg.hideViewmodel && scene === g.viewmodel?.scene) return;
        if (!tp.inRender && !g.postfx && scene === g.scene && camera === g.camera) {
          return thirdPersonWorldRender(g,
            () => original.call(this, scene, camera, ...args));
        }
        return original.call(this, scene, camera, ...args);
      };
      tpRendererHooks.add(renderer);
    }
  }

  setInterval(ensureThirdPersonHooks, 100);

  // =====================================================
  // AUTO STRAFE - NATIVE MOVEMENT / AIR ACCELERATION
  // =====================================================

  // The game's player.update() reads its action binds with input.downA(),
  // consumes mouse deltas with input.consumeMouse(), and accelerates with
  // player.accelerate(wishX, wishZ, wishSpeed, acceleration, deltaTime).
  // Wrap these only DURING the player's own update: ground movement, ladders,
  // custom keybinds, camera input and unrelated systems stay untouched.
  // Original SillyHack WASD Autostrafe v1.2 steering math.
  // Compare both acceleration headings against the actual WASD target
  // and choose the one that makes the resulting velocity point closest
  // to the player's desired movement direction.
  let autoStrafeLastSide = 1;

  function strafeDirection(player, input, cap, acceleration, delta) {
    const forward = Number(!!input.downA('forward')) -
      Number(!!input.downA('back'));
    const right = Number(!!input.downA('right')) -
      Number(!!input.downA('left'));
    if (!forward && !right) return null;

    const sin = Math.sin(player.yaw);
    const cos = Math.cos(player.yaw);
    let targetX = -sin * forward + cos * right;
    let targetZ = -cos * forward - sin * right;
    const targetLength = Math.hypot(targetX, targetZ);
    if (!(targetLength > 0) || !(cap > 0) ||
        !(acceleration > 0) || !(delta > 0)) return null;

    targetX /= targetLength;
    targetZ /= targetLength;

    const vx = Number(player.vx) || 0;
    const vz = Number(player.vz) || 0;
    const speed = Math.hypot(vx, vz);
    if (speed < 0.05) return [targetX, targetZ];

    const step = Math.min(acceleration * delta, cap);
    const dot = Math.max(0, cap - step);
    const angle = Math.acos(Math.min(1, dot / speed));
    const velocityAngle = Math.atan2(vz, vx);
    const targetAngle = Math.atan2(targetZ, targetX);

    let best = null;
    for (const sign of [autoStrafeLastSide, -autoStrafeLastSide]) {
      const heading = velocityAngle + sign * angle;
      const x = Math.cos(heading);
      const z = Math.sin(heading);
      const amount = Math.max(0, Math.min(
        acceleration * delta, cap - (vx * x + vz * z)
      ));
      const nextAngle = Math.atan2(vz + z * amount, vx + x * amount);
      const error = Math.abs(Math.atan2(
        Math.sin(nextAngle - targetAngle),
        Math.cos(nextAngle - targetAngle)
      ));

      if (!best || error < best.error - 1e-9) {
        best = { x, z, side: sign, error };
      }
    }

    if (!best) return null;
    autoStrafeLastSide = best.side;
    return [best.x, best.z];
  }

  // Match the old WASD module's default 100% air speed limiter.
  // This applies only to WASD Autostrafe; Legit mode remains untouched.
  // Optional WASD-only speed cap. With the custom adjustment disabled,
  // the old 100% cap remains in effect exactly as in v1.6.5.
  function limitWASDAirSpeed(player, game) {
    let limit = game.weapons?.maxSpeed?.();
    if (!Number.isFinite(limit) || !(limit > 0)) return;

    const rawPercent = Number(cfg.strafeSpeedPercent);
    const customPercent = Number.isFinite(rawPercent)
      ? Math.max(50, Math.min(150, rawPercent)) : 100;
    limit *= (cfg.strafeCustomSpeed ? customPercent : 100) / 100;

    if (player.crouching) limit *= 0.34;
    else if (player.walking) limit *= 0.52;

    if (Number.isFinite(player.tagFactor)) {
      limit *= Math.max(0, Math.min(1, player.tagFactor));
    }

    const speed = Math.hypot(player.vx, player.vz);
    if (speed > limit && speed > 0) {
      const scale = limit / speed;
      player.vx *= scale;
      player.vz *= scale;
    }
  }

  let legitLockedEnemy=null;
  let legitReactionEnemy=null,legitReactionAt=0;
  const legitRcsStates=new WeakMap();

  function applyLegitRcs(player,game,input) {
    const weapons=game.weapons;
    if (!weapons) return;
    let state=legitRcsStates.get(player);
    if (!state) {state={pitch:0,yaw:0};legitRcsStates.set(player,state);}
    const punchPitch=(Number(weapons.punchP)||0)*Math.PI/180;
    const punchYaw=(Number(weapons.punchY)||0)*Math.PI/180;
    if (cfg.legitRcs && !cfg.noRecoil && input?.mouse1 &&
        game.state==='playing' && player.alive && !uiOpen) {
      const amount=Math.max(0,Math.min(100,Number(cfg.legitRcsPercent)||0))/100;
      player.pitch-=(punchPitch-state.pitch)*amount;
      player.yaw-=(punchYaw-state.yaw)*amount;
    }
    state.pitch=punchPitch;
    state.yaw=punchYaw;
  }

  function applyLegitAimbot(player, game, dt, input) {
    if (!cfg.legitTargetLock) legitLockedEnemy=null;
    if (!cfg.legitAimbot || cfg.rageBot || cfg.silentAim || uiOpen ||
        game.state !== 'playing' || !player.alive || game.frozen ||
        (cfg.legitAimMode === 'fire' && !input?.mouse1)) return;
    const def = game.weapons?.def?.();
    if (!firearmClasses.has(def?.class)) return;

    const eye = {
      x:Number(player.x),
      y:Number(player.y)+Number(player.eyeH??1.6256)-Number(player._stepLift||0),
      z:Number(player.z)
    };
    const pitch=Number(player.pitch), yaw=Number(player.yaw);
    if (![eye.x,eye.y,eye.z,pitch,yaw].every(Number.isFinite)) return;
    const cp=Math.cos(pitch);
    const forward={x:-Math.sin(yaw)*cp,y:Math.sin(pitch),z:-Math.cos(yaw)*cp};
    const fov=Math.max(.5,Math.min(30,Number(cfg.legitAimFov)||5))*Math.PI/180;
    const maxDistance=Math.max(10,Math.min(300,Number(cfg.legitAimMaxDistance)||150));
    let best=null,lockedBest=null;

    for (const enemy of characterEntities(game)) {
      if (!enemy?.alive || enemy.team===player.team || enemy.spawnProt>0) continue;
      const agent=characterAgent(enemy);
      if (!agent?.bones) continue;
      agent.root?.updateMatrixWorld?.(true);
      const normalized=new Map(Object.entries(agent.bones).map(
        ([name,bone])=>[name.toLowerCase(),bone]
      ));
      const pointFor=(name,localPoint)=>{
        const bone=agent.bones[name]||normalized.get(name.toLowerCase());
        bone?.updateWorldMatrix?.(true,false);
        const m=bone?.matrixWorld?.elements;
        if (!m) return null;
        return localPoint ? rageWorldPoint(m,localPoint) : {x:m[12],y:m[13],z:m[14]};
      };
      let points;
      if (cfg.legitAimBone==='chest') points=[pointFor('spine_2',[4.8,.15,0])];
      else if (cfg.legitAimBone==='nearest') points=[
        pointFor('head_0',[1.25,1,0]), pointFor('neck_0'),
        pointFor('spine_1'), pointFor('spine_2',[4.8,.15,0]),
        pointFor('spine_3'), pointFor('spine_0'), pointFor('pelvis')
      ];
      else points=[pointFor('head_0',[1.25,1,0])];

      for (const target of points) {
        if (!target) continue;
        const dx=target.x-eye.x,dy=target.y-eye.y,dz=target.z-eye.z;
        const distance=Math.hypot(dx,dy,dz);
        if (!(distance>.01) || distance>maxDistance) continue;
        let visible=false;
        try {
          visible=game.physics?.lineClear?.(eye.x,eye.y,eye.z,target.x,target.y,target.z)===true;
          if (visible && cfg.legitCheckSmoke &&
              typeof game.grenadeMgr?.lineBlockedBySmoke==='function') {
            visible=!game.grenadeMgr.lineBlockedBySmoke(
              eye.x,eye.y,eye.z,target.x,target.y,target.z
            );
          }
        } catch { visible=false; }
        if (!visible) continue;
        const inv=1/distance;
        const dot=Math.max(-1,Math.min(1,
          forward.x*dx*inv+forward.y*dy*inv+forward.z*dz*inv));
        const angle=Math.acos(dot);
        if (angle>fov) continue;
        const candidate={target,angle,enemy};
        if (!best || angle<best.angle) best=candidate;
        if (cfg.legitTargetLock && enemy===legitLockedEnemy &&
            (!lockedBest || angle<lockedBest.angle)) lockedBest=candidate;
      }
    }
    if (lockedBest) best=lockedBest;
    if (!best) {
      if (cfg.legitReactionDelay) {legitReactionEnemy=null;legitReactionAt=0;}
      return;
    }
    if (cfg.legitTargetLock) legitLockedEnemy=best.enemy;
    if (cfg.legitReactionDelay) {
      if (legitReactionEnemy!==best.enemy) {
        legitReactionEnemy=best.enemy;legitReactionAt=performance.now();return;
      }
      const delay=Math.max(0,Math.min(1000,Number(cfg.legitReactionMs)||0));
      if (performance.now()-legitReactionAt<delay) return;
    } else {
      legitReactionEnemy=null;legitReactionAt=0;
    }
    const deadzone=cfg.legitDeadzone
      ? Math.max(0,Math.min(5,Number(cfg.legitDeadzoneFov)||0))*Math.PI/180
      : 0;
    if (best.angle<=deadzone) return;
    const dx=best.target.x-eye.x,dy=best.target.y-eye.y,dz=best.target.z-eye.z;
    let targetYaw=Math.atan2(-dx,-dz);
    let targetPitch=Math.atan2(dy,Math.hypot(dx,dz));
    if (cfg.legitHumanize) {
      const amount=Math.max(0,Math.min(2,Number(cfg.legitHumanizeAmount)||0))*Math.PI/180;
      const phase=performance.now()/700+(Number(best.enemy?.x)||0);
      targetYaw+=Math.sin(phase)*amount;
      targetPitch+=Math.cos(phase*.83)*amount*.65;
    }
    const yawDelta=Math.atan2(Math.sin(targetYaw-player.yaw),Math.cos(targetYaw-player.yaw));
    const pitchDelta=targetPitch-player.pitch;
    const frame=Math.min(.05,Math.max(0,Number(dt)||0));
    let smooth=Math.max(1,Math.min(30,Number(cfg.legitSmooth)||8));
    if (cfg.legitDynamicSmooth) {
      smooth=Math.min(30,smooth+(1-Math.min(1,best.angle/fov))*12);
    }
    const alpha=1-Math.exp(-(31-smooth)*frame);
    const maxStep=Math.max(30,Math.min(720,
      Number(cfg.legitMaxSpeed)||180))*Math.PI/180*frame;
    player.yaw+=Math.max(-maxStep,Math.min(maxStep,yawDelta*alpha));
    player.pitch+=Math.max(-maxStep,Math.min(maxStep,pitchDelta*alpha));
    const limit=Math.PI/2-.01;
    player.pitch=Math.max(-limit,Math.min(limit,player.pitch));
  }

  function applyMovementFeatures(player, game, dt, input, wasGrounded, wasLadder) {
    if (cfg.lowGravity && !player.onGround && !player.onLadder && Number.isFinite(dt)) {
      const gravity = Math.max(0, Math.min(100, Number(cfg.gravityPercent) || 0));
      player.vy += 20.32 * (1 - gravity / 100) * dt;
    }
    if (cfg.jumpBoost && wasGrounded && !wasLadder &&
        !player.onGround && !player.onLadder && player.vy > 0) {
      const jump = Math.max(100, Math.min(300, Number(cfg.jumpBoostPercent) || 140));
      player.vy *= jump / 100;
    }
    if ((cfg.groundSpeed || (cfg.crouchBoost && player.crouching)) &&
        player.onGround && !player.onLadder &&
        typeof input?.downA === 'function') {
      const moving = input.downA('forward') || input.downA('back') ||
        input.downA('left') || input.downA('right');
      if (moving) {
        const base = Number(game.weapons?.maxSpeed?.()) || 6.35;
        let target = base;
        if (cfg.groundSpeed) {
          target *= Math.max(100, Math.min(300,
            Number(cfg.groundSpeedPercent) || 150)) / 100;
        }
        if (player.crouching) {
          const nativeCrouch = target * .34;
          const boostedCrouch = cfg.crouchBoost
            ? base * Math.max(100, Math.min(300,
              Number(cfg.crouchSpeedPercent) || 180)) / 100
            : 0;
          target = Math.max(nativeCrouch, boostedCrouch);
        }
        else if (player.walking) target *= .52;
        if (Number.isFinite(player.tagFactor)) {
          target *= Math.max(.35, Math.min(1, player.tagFactor));
        }
        const speed = Math.hypot(Number(player.vx) || 0, Number(player.vz) || 0);
        if (speed > .01 && speed < target) {
          const scale = target / speed;
          player.vx *= scale;
          player.vz *= scale;
        }
      }
    }
  }

  function ensureAutoStrafeHook() {
    const g = window.game;
    const player = g?.player;
    if (!player || typeof player.update !== 'function') return;
    if (player.update.__amnesisAutoStrafe) return;
    const originalUpdate = player.update;

    function patchedPlayerUpdate(dt, input, ...rest) {
      syncRageLifecycle(g);
      const rageMoveTarget=rageCombatActive(g,true)&&rageLastTarget?.enemy?.alive&&
        rageLastTarget.enemy.team!==this.team;
      const runUpdate = () => {
        const wasGrounded = !!this.onGround;
        const wasLadder = !!this.onLadder;
        const beforeVx=Number(this.vx)||0,beforeVz=Number(this.vz)||0;
        const oldForceCrouch=this.forceCrouch;
        const oldDownA = input?.downA;
        const hadDownA = !!input && Object.prototype.hasOwnProperty.call(input, 'downA');
        const oldLadderMove=this.ladderMove;
        const hadLadderMove=Object.prototype.hasOwnProperty.call(this,'ladderMove');
        const oldAccelerate=this.accelerate;
        const hadAccelerate=Object.prototype.hasOwnProperty.call(this,'accelerate');
        let inputDirectionWrapped = false;
        if ((cfg.autoAirCrouch && !this.onGround && !this.onLadder) ||
            (cfg.rageAutoCrouch && cfg.rageBot && rageMoveTarget) ||
            (cfg.autoLandCrouch&&Number(g.time)<Number(this._amnesisLandCrouchUntil||0))) {
          this.forceCrouch=true;
        }
        if (cfg.rageAutoJump&&cfg.rageBot&&rageMoveTarget&&this.onGround&&
            !this.onLadder) this.jumpBuffer=.12;
        if (cfg.reverseMovement&&typeof oldDownA==='function') {
          const swaps={forward:'back',back:'forward',left:'right',right:'left'};
          input.downA=function(action,...args) {
            return oldDownA.call(this,swaps[action]||action,...args);
          };
          inputDirectionWrapped=true;
        }
        if (cfg.autoWalk && typeof input?.downA === 'function') {
          const activeDownA=input.downA;
          input.downA = function(action, ...args) {
            if (action === 'forward' && !activeDownA.call(this, 'back')) return true;
            return activeDownA.call(this, action, ...args);
          };
          inputDirectionWrapped = true;
        }
        if (cfg.ladderBoost && typeof oldLadderMove==='function') {
          this.ladderMove=function(...args) {
            const active=oldLadderMove.apply(this,args);
            if (active&&this.onLadder) {
              const factor=Math.max(1,Math.min(3,
                Number(cfg.ladderSpeedPercent)||160)/100);
              this.vx*=factor;this.vy*=factor;this.vz*=factor;
            }
            return active;
          };
        }
        if (cfg.airAccelBoost && typeof oldAccelerate==='function') {
          this.accelerate=function(x,z,wishSpeed,acceleration,delta) {
            if (!this.onGround&&!this.onLadder) {
              acceleration*=Math.max(1,Math.min(3,
                Number(cfg.airAccelPercent)||160)/100);
            }
            return oldAccelerate.call(this,x,z,wishSpeed,acceleration,delta);
          };
        }
        try {
          const result = originalUpdate.call(this, dt, input, ...rest);
          applyLegitRcs(this,g,input);
          applyLegitAimbot(this, g, dt, input);
          if (cfg.edgeJump&&wasGrounded&&!wasLadder&&!this.onGround&&
              !this.onLadder&&this.vy<=0) {
            this.vy=7.6706317758*((80-(Number(this.stamina)||0))/80);
            this.stamina=Math.min(80,(Number(this.stamina)||0)+6.4);
            this._jumpSnapAt=(Number(g.time)||0)+.1;
            this._jumpSnap=null;
          }
          if (cfg.momentumPreserve&&!wasGrounded&&this.onGround) {
            const beforeSpeed=Math.hypot(beforeVx,beforeVz);
            const afterSpeed=Math.hypot(Number(this.vx)||0,Number(this.vz)||0);
            const dot=beforeSpeed>.01&&afterSpeed>.01?
              (beforeVx*this.vx+beforeVz*this.vz)/(beforeSpeed*afterSpeed):0;
            if (dot>.9&&afterSpeed<beforeSpeed) {
              const scale=beforeSpeed/afterSpeed;
              this.vx*=scale;this.vz*=scale;
            }
          }
          if (cfg.autoLandCrouch&&!wasGrounded&&this.onGround) {
            this._amnesisLandCrouchUntil=(Number(g.time)||0)+
              Math.max(50,Math.min(1000,Number(cfg.landCrouchTime)||250))/1000;
          }
          applyMovementFeatures(this, g, dt, input, wasGrounded, wasLadder);
          applySuperBhopBoost(this, g);
          if (cfg.fallSpeedLimiter&&!this.onGround&&!this.onLadder) {
            this.vy=Math.max(Number(this.vy)||0,-Math.max(2,Math.min(30,
              Number(cfg.maxFallSpeed)||8)));
          }
          if (cfg.airBrake && !this.onGround && !this.onLadder) {
            const moving = typeof input?.downA === 'function' &&
              (input.downA('forward') || input.downA('back') ||
               input.downA('left') || input.downA('right'));
            if (!moving) {
              const strength = Math.max(10, Math.min(100,
                Number(cfg.airBrakeStrength) || 85)) / 100;
              const factor = Math.pow(1 - strength, Math.max(0, Number(dt) || 0) * 60);
              this.vx *= factor;
              this.vz *= factor;
            }
          }
          return result;
        } finally {
          this.forceCrouch=oldForceCrouch;
          if (cfg.ladderBoost) {
            if (hadLadderMove) this.ladderMove=oldLadderMove;
            else delete this.ladderMove;
          }
          if (cfg.airAccelBoost) {
            if (hadAccelerate) this.accelerate=oldAccelerate;
            else delete this.accelerate;
          }
          if (inputDirectionWrapped) {
            if (hadDownA) input.downA = oldDownA;
            else delete input.downA;
          }
        }
      };
      const mode = ['legit', 'rage'].includes(cfg.strafeMode) ? cfg.strafeMode : 'wasd';
      if (!cfg.autoStrafe || !input || !this.alive ||
          g.state !== 'playing' || g.frozen || uiOpen ||
          this.onLadder) {
        return runUpdate();
      }

      if (mode === 'legit' &&
          typeof input.downA === 'function' &&
          typeof input.consumeMouse === 'function') {
        const oldDownA = input.downA;
        const oldConsumeMouse = input.consumeMouse;
        const hadDownA = Object.prototype.hasOwnProperty.call(input, 'downA');
        const hadConsume = Object.prototype.hasOwnProperty.call(input, 'consumeMouse');
        let mouseSide = 0;

        input.consumeMouse = function(...args) {
          const motion = oldConsumeMouse.apply(this, args);
          const dx = Number(motion?.dx) || 0;
          mouseSide = dx > 0.1 ? 1 : dx < -0.1 ? -1 : 0;
          return motion;
        };
        input.downA = function(action, ...args) {
          // The game already consumed the same mouse delta for actual look.
          // Only synthesize A/D during airborne movement; ground stays native.
          if (!player.onGround && !player.onLadder && mouseSide &&
              (action === 'left' || action === 'right')) {
            return action === 'right' ? mouseSide > 0 : mouseSide < 0;
          }
          return oldDownA.call(this, action, ...args);
        };
        try {
          return runUpdate();
        } finally {
          if (hadDownA) input.downA = oldDownA;
          else delete input.downA;
          if (hadConsume) input.consumeMouse = oldConsumeMouse;
          else delete input.consumeMouse;
        }
      }

      if (mode === 'rage' && g.input?.locked && spaceHeld &&
          typeof this.accelerate === 'function' &&
          typeof input.downA === 'function') {
        const oldAccelerate = this.accelerate;
        const hadAccelerate = Object.prototype.hasOwnProperty.call(this, 'accelerate');
        this.accelerate = function(x, z, wishSpeed, accel, delta) {
          if (this.onGround || this.onLadder) {
            return oldAccelerate.call(this, x, z, wishSpeed, accel, delta);
          }
          const baseSpeed = g.weapons?.maxSpeed?.();
          if (!Number.isFinite(baseSpeed) || !(baseSpeed > 0)) {
            return oldAccelerate.call(this, x, z, wishSpeed, accel, delta);
          }
          const rawPercent = Number(cfg.rageSpeedPercent);
          const percent = Number.isFinite(rawPercent)
            ? Math.max(50, Math.min(150, rawPercent)) : 100;
          const speed = baseSpeed * percent / 100;
          const forward = Number(!!input.downA('forward')) - Number(!!input.downA('back'));
          const right = Number(!!input.downA('right')) - Number(!!input.downA('left'));
          // No steering oscillation or acceleration: snap to the requested heading.
          // With no directional input, keep the existing heading at the selected speed.
          let dx = -Math.sin(this.yaw) * forward + Math.cos(this.yaw) * right;
          let dz = -Math.cos(this.yaw) * forward - Math.sin(this.yaw) * right;
          if (!forward && !right) {
            dx = Number(this.vx) || 0;
            dz = Number(this.vz) || 0;
          }
          const length = Math.hypot(dx, dz);
          if (length > 0) {
            this.vx = dx / length * speed;
            this.vz = dz / length * speed;
          }
        };
        try {
          return runUpdate();
        } finally {
          if (hadAccelerate) this.accelerate = oldAccelerate;
          else delete this.accelerate;
        }
      }

      if (mode === 'wasd' && g.input?.locked &&
          typeof this.accelerate === 'function' &&
          typeof input.downA === 'function') {
        const oldAccelerate = this.accelerate;
        const hadAccelerate = Object.prototype.hasOwnProperty.call(this, 'accelerate');
        this.accelerate = function(x, z, wishSpeed, accel, delta) {
          if (!this.onGround && !this.onLadder) {
            const desired = strafeDirection(this, input, wishSpeed, accel, delta);
            if (desired) [x, z] = desired;
          }
          const result = oldAccelerate.call(this, x, z, wishSpeed, accel, delta);
          if (!this.onGround && !this.onLadder) {
            limitWASDAirSpeed(this, g);
          }
          return result;
        };
        try {
          return runUpdate();
        } finally {
          if (hadAccelerate) this.accelerate = oldAccelerate;
          else delete this.accelerate;
        }
      }
      return runUpdate();
    }

    Object.defineProperty(patchedPlayerUpdate, '__amnesisAutoStrafe', {
      value: true
    });
    player.update = patchedPlayerUpdate;
  }

  setInterval(ensureAutoStrafeHook, 100);

  // =====================================================
  // SKELETON ESP - UNCAPPED
  // =====================================================

  const skeleton = [
    ['head_0', 'neck_0'],
    ['neck_0', 'spine_3'],
    ['spine_3', 'spine_2'],
    ['spine_2', 'spine_1'],
    ['spine_1', 'spine_0'],
    ['spine_0', 'pelvis'],

    ['spine_3', 'clavicle_L'],
    ['clavicle_L', 'arm_upper_L'],
    ['arm_upper_L', 'arm_lower_L'],
    ['arm_lower_L', 'hand_L'],

    ['spine_3', 'clavicle_R'],
    ['clavicle_R', 'arm_upper_R'],
    ['arm_upper_R', 'arm_lower_R'],
    ['arm_lower_R', 'hand_R'],

    ['pelvis', 'leg_upper_L'],
    ['leg_upper_L', 'leg_lower_L'],
    ['leg_lower_L', 'ankle_L'],

    ['pelvis', 'leg_upper_R'],
    ['leg_upper_R', 'leg_lower_R'],
    ['leg_lower_R', 'ankle_R']
  ];

  const fallback = {
    spine_0: ['spine_1'],
    clavicle_L: ['arm_upper_L'],
    clavicle_R: ['arm_upper_R'],
    ankle_L: ['foot_L'],
    ankle_R: ['foot_R']
  };

  let overlay, ctx;

  // =====================================================
  // ARROW ESP - ENEMY BEARING + TRUE WORLD LINE OF SIGHT
  // =====================================================
  // Show enemy bearings on a small ring around the crosshair, including
  // enemies outside the view. Neutral = occluded, red = unobstructed.
  // Do not use the rendered skeleton orientation: Anti Aim may rotate it.
  const arrowSightCache = new WeakMap();

  function arrowEnemies(g) {
    // allEntities() also includes multiplayer players, not only bots.
    try {
      if (typeof g.allEntities === 'function') return [...g.allEntities()];
    } catch {}
    const raw = g.botMgr?.bots;
    return Array.isArray(raw) ? raw : Object.values(raw || {});
  }

  function arrowVisible(g, enemy, now) {
    const old = arrowSightCache.get(enemy);
    if (old && now - old.time < 80) return old.visible;

    const p = g.player;
    const physics = g.physics;
    let visible = false;
    if (typeof physics?.lineClear === 'function') {
      // Use physical player eye coordinates, not the chase-camera offset.
      const sx = Number(p.x);
      const sy = Number(p.y) + Number(p.eyeH ?? 1.6256) -
        Number(p._stepLift || 0);
      const sz = Number(p.z);
      const ex = Number(enemy.x);
      const ez = Number(enemy.z);
      const ey = Number(enemy.y);
      const eye = Number(enemy.eyeH ?? (enemy.crouching ? 1.1 : 1.6256));
      const height = Number(enemy.height ?? (enemy.crouching ? 1.15 : 1.8));
      const smoke = g.grenadeMgr;

      if ([sx, sy, sz, ex, ez, ey, eye, height].every(Number.isFinite)) {
        // A partial peek counts as visible if any head/torso point is clear.
        for (const ty of [ey + eye, ey + height * .68, ey + height * .40]) {
          try {
            if (!physics.lineClear(sx, sy, sz, ex, ty, ez)) continue;
            if (typeof smoke?.lineBlockedBySmoke === 'function' &&
                smoke.lineBlockedBySmoke(sx, sy, sz, ex, ty, ez)) continue;
            visible = true;
            break;
          } catch { /* Unknown trace is not assumed visible. */ }
        }
      }
    }
    arrowSightCache.set(enemy, {time:now, visible});
    return visible;
  }

  function drawArrowESP(g, w, h) {
    const p = g.player;
    const yaw = Number(p?.yaw);
    if (!p?.alive || !Number.isFinite(yaw) || !ctx) return;

    const half = Math.min(w, h) / 2;
    const radiusPercent = Math.max(25, Math.min(90,
      Number(cfg.arrowRadiusPercent) || 54));
    const radius = Math.min(Math.max(30, half - 20),
      Math.max(30, half * radiusPercent / 100));
    const size = Math.max(8, Math.min(30, Number(cfg.arrowSize) || 15));
    const cx = w / 2, cy = h / 2;
    const sinYaw = Math.sin(yaw), cosYaw = Math.cos(yaw);
    const now = performance.now();
    const targets = [];

    for (const enemy of arrowEnemies(g)) {
      if (!enemy || enemy === p || !enemy.alive ||
          (p.team != null && enemy.team === p.team)) continue;
      const dx = Number(enemy.x) - Number(p.x);
      const dz = Number(enemy.z) - Number(p.z);
      const distance = Math.hypot(dx, dz);
      if (!Number.isFinite(distance) || distance < .04) continue;

      // yaw=0 faces -Z: forward -> top, right -> right, back -> bottom.
      const forward = -sinYaw * dx - cosYaw * dz;
      const right = cosYaw * dx - sinYaw * dz;
      const angle = Math.atan2(right, forward);
      targets.push({enemy, distance, angle});
    }

    // Nearest arrows render last when multiple enemies share one bearing.
    targets.sort((a, b) => b.distance - a.distance);
    ctx.save();
    ctx.lineJoin = 'round';
    ctx.lineWidth = 2;
    ctx.strokeStyle = 'rgba(0,0,0,.92)';
    ctx.shadowColor = 'rgba(0,0,0,.75)';
    ctx.shadowBlur = 4;

    for (const target of targets) {
      const {angle, enemy, distance} = target;
      const vx = Math.sin(angle), vy = -Math.cos(angle);
      const tipX = cx + vx * radius;
      const tipY = cy + vy * radius;
      const baseX = tipX - vx * size;
      const baseY = tipY - vy * size;
      const perpendicularX = -vy, perpendicularY = vx;

      ctx.beginPath();
      ctx.moveTo(tipX, tipY);
      ctx.lineTo(baseX + perpendicularX * size * .47, baseY + perpendicularY * size * .47);
      ctx.lineTo(baseX - perpendicularX * size * .47, baseY - perpendicularY * size * .47);
      ctx.closePath();
      ctx.fillStyle = arrowVisible(g, enemy, now)
        ? cfg.arrowVisibleColor : cfg.arrowHiddenColor;
      ctx.fill();
      ctx.stroke();
      if (cfg.arrowDistance) {
        ctx.save();
        ctx.font='bold 9px Consolas,monospace';
        ctx.textAlign='center';
        ctx.textBaseline='middle';
        ctx.lineWidth=3;
        const tx=baseX-vx*8,ty=baseY-vy*8;
        ctx.strokeStyle='rgba(0,0,0,.9)';
        ctx.strokeText(Math.round(distance)+'m',tx,ty);
        ctx.fillStyle=cfg.espTextColor;
        ctx.fillText(Math.round(distance)+'m',tx,ty);
        ctx.restore();
      }
    }
    ctx.restore();
  }

  function drawRadarESP(g) {
    const player=g.player;
    if (!player?.alive||!ctx) return;
    const size=Math.max(80,Math.min(220,Number(cfg.radarSize)||130));
    const range=Math.max(10,Math.min(200,Number(cfg.radarRange)||60));
    const x=16,y=16,cx=x+size/2,cy=y+size/2,r=size/2-8;
    const yaw=Number(player.yaw)||0,sin=Math.sin(yaw),cos=Math.cos(yaw);
    ctx.save();
    ctx.fillStyle='rgba(8,10,13,.72)';
    ctx.strokeStyle='rgba(255,255,255,.16)';
    ctx.lineWidth=1;
    ctx.fillRect(x,y,size,size);ctx.strokeRect(x,y,size,size);
    ctx.beginPath();ctx.moveTo(cx,y+5);ctx.lineTo(cx,y+size-5);
    ctx.moveTo(x+5,cy);ctx.lineTo(x+size-5,cy);ctx.stroke();
    ctx.fillStyle=cfg.accent;
    ctx.beginPath();ctx.moveTo(cx,cy-5);ctx.lineTo(cx-4,cy+4);ctx.lineTo(cx+4,cy+4);ctx.closePath();ctx.fill();
    for (const enemy of characterEntities(g)) {
      if (!enemy?.alive) continue;
      const dx=Number(enemy.x)-Number(player.x),dz=Number(enemy.z)-Number(player.z);
      if (![dx,dz].every(Number.isFinite)) continue;
      const right=cos*dx-sin*dz,forward=-sin*dx-cos*dz;
      const scale=r/range;
      let rx=right*scale,ry=-forward*scale;
      const length=Math.hypot(rx,ry);
      if (length>r) {rx*=r/length;ry*=r/length;}
      ctx.fillStyle=enemy.team===player.team?cfg.espTeamColor:cfg.espEnemyColor;
      ctx.beginPath();ctx.arc(cx+rx,cy+ry,3,0,Math.PI*2);ctx.fill();
    }
    ctx.restore();
  }

  function mul(m, a) {
    return [
      m[0]*a[0] + m[4]*a[1] + m[8]*a[2] + m[12]*a[3],
      m[1]*a[0] + m[5]*a[1] + m[9]*a[2] + m[13]*a[3],
      m[2]*a[0] + m[6]*a[1] + m[10]*a[2] + m[14]*a[3],
      m[3]*a[0] + m[7]*a[1] + m[11]*a[2] + m[15]*a[3]
    ];
  }

  function point(bone, view, projection, w, h) {
    const m = bone?.matrixWorld?.elements;
    if (!m) return null;

    const eye = mul(view, [m[12], m[13], m[14], 1]);
    const clip = mul(projection, eye);

    if (clip[3] <= 0.001 ||
        clip[2] < -clip[3] ||
        clip[2] > clip[3]) return null;

    const x = (clip[0] / clip[3] * 0.5 + 0.5) * w;
    const y = (0.5 - clip[1] / clip[3] * 0.5) * h;

    return Number.isFinite(x) && Number.isFinite(y) ? [x, y] : null;
  }

  function espFrame() {
    // No 16 ms throttle. Render on every browser animation frame.
    requestAnimationFrame(espFrame);

    const g = window.game;
    const entityOverlay = !!(cfg.boneESP || cfg.boxESP || cfg.cornerBoxESP || cfg.boxFill ||
      cfg.headDotESP || cfg.espDistance || cfg.espArmor || cfg.espHealth ||
      cfg.espHealthText || cfg.espName || cfg.espWeapon || cfg.espTracers);
    const active = !!(entityOverlay || cfg.arrowESP || cfg.radarESP || cfg.crosshairEnabled || cfg.velocityHud ||
        (cfg.legitAimbot && cfg.legitAimCircle) ||
        (cfg.silentAim && cfg.silentCircle)) &&
      g?.state === 'playing' &&
      g?.camera &&
      (g?.botMgr?.bots || typeof g?.allEntities === 'function');

    if (!overlay) return;

    overlay.style.display = active ? 'block' : 'none';
    if (!active) return;

    const canvas = document.querySelector('#game canvas') || g.renderer?.domElement;
    if (!canvas) return;

    const bounds = canvas.getBoundingClientRect();
    if (bounds.width <= 0 || bounds.height <= 0) return;

    overlay.style.left = `${bounds.left}px`;
    overlay.style.top = `${bounds.top}px`;
    overlay.style.width = `${bounds.width}px`;
    overlay.style.height = `${bounds.height}px`;

    const dpr = Math.min(window.devicePixelRatio || 1, 2);
    const pw = Math.round(bounds.width * dpr);
    const ph = Math.round(bounds.height * dpr);

    if (overlay.width !== pw || overlay.height !== ph) {
      overlay.width = pw;
      overlay.height = ph;
    }

    ctx.setTransform(dpr, 0, 0, dpr, 0, 0);
    ctx.clearRect(0, 0, bounds.width, bounds.height);

    if (cfg.velocityHud && g.player) {
      const speed = Math.hypot(Number(g.player.vx) || 0, Number(g.player.vz) || 0);
      const text = Math.round(speed / .0254) + ' u/s  |  ' + speed.toFixed(1) + ' m/s';
      ctx.save();
      ctx.font = 'bold 13px Consolas,monospace';
      ctx.textAlign = 'center';
      ctx.lineWidth = 4;
      ctx.strokeStyle = 'rgba(0,0,0,.9)';
      ctx.strokeText(text, bounds.width / 2, bounds.height - 42);
      ctx.fillStyle = cfg.accent;
      ctx.fillText(text, bounds.width / 2, bounds.height - 42);
      ctx.restore();
    }

    if (cfg.crosshairEnabled) {
      const cx = bounds.width / 2;
      const cy = bounds.height / 2;
      const size = Math.max(2, Math.min(30, Number(cfg.crosshairSize) || 8));
      const gap = Math.max(0, Math.min(20, Number(cfg.crosshairGap) || 0));
      ctx.save();
      ctx.beginPath();
      ctx.moveTo(cx - gap - size, cy); ctx.lineTo(cx - gap, cy);
      ctx.moveTo(cx + gap, cy); ctx.lineTo(cx + gap + size, cy);
      ctx.moveTo(cx, cy - gap - size); ctx.lineTo(cx, cy - gap);
      ctx.moveTo(cx, cy + gap); ctx.lineTo(cx, cy + gap + size);
      ctx.strokeStyle = cfg.crosshairColor;
      ctx.lineWidth = Math.max(1, Math.min(6, Number(cfg.crosshairWidth) || 2));
      ctx.lineCap = 'square';
      ctx.shadowColor = 'rgba(0,0,0,.9)';
      ctx.shadowBlur = 2;
      if (cfg.crosshairOutline) {
        ctx.strokeStyle='rgba(0,0,0,.95)';
        ctx.lineWidth=Math.max(1,Math.min(6,Number(cfg.crosshairWidth)||2))+2;
        ctx.stroke();
        ctx.strokeStyle=cfg.crosshairColor;
        ctx.lineWidth=Math.max(1,Math.min(6,Number(cfg.crosshairWidth)||2));
      }
      ctx.stroke();
      if (cfg.crosshairDot) {
        ctx.fillStyle=cfg.crosshairColor;
        ctx.strokeStyle='rgba(0,0,0,.95)';
        ctx.lineWidth=cfg.crosshairOutline?2:0;
        ctx.beginPath();
        ctx.arc(cx,cy,Math.max(1,Math.min(6,Number(cfg.crosshairDotSize)||2)),0,Math.PI*2);
        ctx.fill();
        if (cfg.crosshairOutline) ctx.stroke();
      }
      ctx.restore();
    }

    if (cfg.silentAim && cfg.silentCircle) {
      ctx.save();
      ctx.beginPath();
      ctx.arc(
        bounds.width / 2,
        bounds.height / 2,
        Number(cfg.silentFov) || 120,
        0,
        Math.PI * 2
      );
      ctx.strokeStyle = cfg.silentCircleColor;
      ctx.globalAlpha = Math.max(.1, Math.min(1,
        Number(cfg.silentCircleOpacity) / 100 || .85));
      ctx.lineWidth = Math.max(.5, Math.min(5,
        Number(cfg.silentCircleWidth) || 1.25));
      ctx.shadowBlur = 0;
      ctx.stroke();
      ctx.restore();
    }

    if (cfg.legitAimbot && cfg.legitAimCircle) {
      const cameraFov=Math.max(1,Number(g.camera?.fov)||73.74)*Math.PI/180;
      const aimFov=Math.max(.5,Math.min(30,Number(cfg.legitAimFov)||5))*Math.PI/180;
      const radius=Math.tan(aimFov)*bounds.height/(2*Math.tan(cameraFov/2));
      ctx.save();
      ctx.beginPath();ctx.arc(bounds.width/2,bounds.height/2,radius,0,Math.PI*2);
      ctx.setLineDash([4,4]);ctx.strokeStyle=cfg.accent;ctx.globalAlpha=.72;
      ctx.lineWidth=1;ctx.stroke();ctx.restore();
    }

    if (cfg.arrowESP) drawArrowESP(g, bounds.width, bounds.height);
    if (cfg.radarESP) drawRadarESP(g);

    if (!entityOverlay) return;

    const camera = (cfg.thirdPerson && g.__amnesisThirdCamera) || g.camera;
    camera.updateMatrixWorld?.(true);

    const view = camera.matrixWorldInverse?.elements;
    const projection = camera.projectionMatrix?.elements;
    if (!view || !projection) return;

    ctx.lineWidth = Math.max(.5, Math.min(6, Number(cfg.espLineWidth) || 2));
    ctx.lineCap = 'round';
    ctx.lineJoin = 'round';
    ctx.shadowColor = '#000';
    ctx.shadowBlur = 3;

    const bots = characterEntities(g);

    for (const bot of bots) {
      if (!bot?.alive || !characterAgent(bot)?.bones) continue;
      if (!cfg.boneESPTeam &&
          g.player?.team != null &&
          bot.team === g.player.team) continue;

      const entityDistance=Math.hypot(
        Number(bot.x)-Number(g.player?.x),
        Number(bot._visY??bot.y)-Number(g.player?.y),
        Number(bot.z)-Number(g.player?.z)
      );
      const maxDistance=Math.max(0,Math.min(300,Number(cfg.espMaxDistance)||0));
      if (!Number.isFinite(entityDistance) ||
          (maxDistance>0 && entityDistance>maxDistance)) continue;

      const agent = characterAgent(bot);
      agent.root?.updateMatrixWorld?.(true);

      const coords = Object.create(null);

      const normalized = new Map(Object.entries(agent.bones).map(([name, bone]) => [name.toLowerCase(), bone]));
      const resolve = name => [name, ...(fallback[name] || [])]
        .map(key => agent.bones[key] || normalized.get(key.toLowerCase())).find(Boolean);

      for (const [a, b] of skeleton) {
        if (!(a in coords)) {
          coords[a] = point(resolve(a), view, projection, bounds.width, bounds.height);
        }
        if (!(b in coords)) {
          coords[b] = point(resolve(b), view, projection, bounds.width, bounds.height);
        }
      }

      const teammate = g.player?.team != null && bot.team === g.player.team;
      let espColor = teammate ? cfg.espTeamColor : cfg.espEnemyColor;
      if (cfg.rainbowESP && !teammate) {
        const speed = Math.max(10, Math.min(200, Number(cfg.rainbowSpeed) || 60));
        const hue = (performance.now() * speed / 100 + (Number(bot.x) || 0) * 17) % 360;
        espColor = `hsl(${hue} 100% 58%)`;
      }
      const entityAlpha=Math.max(.1,Math.min(1,Number(cfg.espOpacity)/100||1));
      ctx.globalAlpha=entityAlpha;
      if (cfg.boneESP) {
        ctx.strokeStyle = espColor;
        ctx.beginPath();
        let count = 0;
        for (const [a, b] of skeleton) {
          const p = coords[a];
          const q = coords[b];
          if (!p || !q) continue;
          if (Math.hypot(p[0] - q[0], p[1] - q[1]) >
              Math.max(bounds.width, bounds.height) * 0.65) continue;
          ctx.moveTo(p[0], p[1]);
          ctx.lineTo(q[0], q[1]);
          count++;
        }
        if (count) ctx.stroke();

        const jointSize = Math.max(0, Math.min(6, Number(cfg.espJointSize) || 0));
        if (jointSize > 0) {
          ctx.fillStyle = espColor;
          for (const p of Object.values(coords)) {
            if (!p) continue;
            ctx.beginPath();
            ctx.arc(p[0], p[1], jointSize, 0, Math.PI * 2);
            ctx.fill();
          }
        }
      }

      const joints = Object.values(coords).filter(p => p &&
        p[0] >= -bounds.width && p[0] <= bounds.width * 2 &&
        p[1] >= -bounds.height && p[1] <= bounds.height * 2
      );

      if (joints.length < 3) { ctx.globalAlpha=1; continue; }

      const left = Math.min(...joints.map(p => p[0]));
      const right = Math.max(...joints.map(p => p[0]));
      const top = Math.min(...joints.map(p => p[1]));
      const bottom = Math.max(...joints.map(p => p[1]));

      const width = right - left;
      const height = bottom - top;

      if (height < 5 ||
          height > bounds.height * 1.5 ||
          width > bounds.width) { ctx.globalAlpha=1; continue; }

      const center = (left + right) / 2;

      if (cfg.boxFill || cfg.boxESP) {
        const boxWidth=Math.max(.5,Math.min(6,Number(cfg.boxThickness)||2));
        ctx.save();
        if (cfg.boxFill) {
          ctx.fillStyle = espColor;
          ctx.globalAlpha = entityAlpha*Math.max(.01, Math.min(.5,
            Number(cfg.boxFillOpacity) / 100 || .12));
          ctx.fillRect(left, top, width, height);
        }
        if (cfg.boxESP) {
          ctx.globalAlpha = entityAlpha;
          ctx.lineWidth = boxWidth+2;
          ctx.strokeStyle = 'rgba(0,0,0,.85)';
          ctx.strokeRect(left, top, width, height);
          ctx.lineWidth = boxWidth;
          ctx.strokeStyle = espColor;
          ctx.strokeRect(left, top, width, height);
        }
        ctx.restore();
      }

      if (cfg.cornerBoxESP) {
        const boxWidth=Math.max(.5,Math.min(6,Number(cfg.boxThickness)||2));
        const corner = Math.max(4, Math.min(width, height) * .22);
        const drawCorners = () => {
          ctx.beginPath();
          ctx.moveTo(left, top + corner); ctx.lineTo(left, top); ctx.lineTo(left + corner, top);
          ctx.moveTo(right - corner, top); ctx.lineTo(right, top); ctx.lineTo(right, top + corner);
          ctx.moveTo(left, bottom - corner); ctx.lineTo(left, bottom); ctx.lineTo(left + corner, bottom);
          ctx.moveTo(right - corner, bottom); ctx.lineTo(right, bottom); ctx.lineTo(right, bottom - corner);
          ctx.stroke();
        };
        ctx.save();
        ctx.lineCap = 'square';
        ctx.lineWidth = boxWidth+2;
        ctx.strokeStyle = 'rgba(0,0,0,.9)';
        drawCorners();
        ctx.lineWidth = boxWidth;
        ctx.strokeStyle = espColor;
        drawCorners();
        ctx.restore();
      }

      if (cfg.headDotESP && coords.head_0) {
        ctx.save();
        ctx.fillStyle = espColor;
        ctx.strokeStyle = 'rgba(0,0,0,.9)';
        ctx.lineWidth = 2;
        ctx.beginPath();
        ctx.arc(coords.head_0[0], coords.head_0[1],
          Math.max(1, Math.min(8, Number(cfg.headDotSize) || 3)), 0, Math.PI * 2);
        ctx.fill();
        ctx.stroke();
        ctx.restore();
      }

      if (cfg.espTracers) {
        ctx.save();
        ctx.strokeStyle = espColor;
        ctx.globalAlpha = entityAlpha*.8;
        ctx.beginPath();
        ctx.moveTo(bounds.width / 2, bounds.height);
        ctx.lineTo(center, bottom);
        ctx.stroke();
        ctx.restore();
      }

      // HEALTH BAR
      if (cfg.espHealth) {
        const health = Number(
          bot.health ?? bot.hp ?? bot.stats?.health ?? bot.player?.health
        );
        const maxHealth = Number(
          bot.maxHealth ?? bot.maxHp ?? bot.stats?.maxHealth ?? 100
        );

        if (Number.isFinite(health) &&
            Number.isFinite(maxHealth) &&
            maxHealth > 0) {
          const fraction = Math.max(0, Math.min(1, health / maxHealth));
          const barHeight = Math.max(12, bottom - top);
          const x = left - 10;
          const y = top;

          ctx.shadowBlur = 0;
          ctx.fillStyle = 'rgba(0,0,0,.8)';
          ctx.fillRect(x - 1, y - 1, 6, barHeight + 2);

          ctx.fillStyle = cfg.espHealthColor;
          ctx.fillRect(
            x,
            y + barHeight * (1 - fraction),
            4,
            barHeight * fraction
          );
          ctx.shadowBlur = 3;
        }
      }

      if (cfg.espArmor) {
        const armor = Math.max(0, Math.min(100, Number(bot.armor) || 0));
        const barHeight = Math.max(12, bottom - top);
        const x = right + 6;
        ctx.save();
        ctx.shadowBlur = 0;
        ctx.fillStyle = 'rgba(0,0,0,.8)';
        ctx.fillRect(x - 1, top - 1, 6, barHeight + 2);
        ctx.fillStyle = bot.helmet ? cfg.espHelmetColor : cfg.espArmorColor;
        ctx.fillRect(x, top + barHeight * (1 - armor / 100), 4,
          barHeight * armor / 100);
        ctx.restore();
      }

      // PLAYER NAME / HELD WEAPON
      if (cfg.espName || cfg.espWeapon || cfg.espDistance || cfg.espHealthText) {
        ctx.save();
        ctx.shadowBlur = 3;
        const fontSize=Math.max(8,Math.min(24,Number(cfg.espFontSize)||12));
        ctx.font = `bold ${fontSize}px Arial,sans-serif`;
        ctx.textAlign = 'center';
        ctx.textBaseline = 'top';

        let y = bottom + 5;

        const label = value => {
          const text = String(value).slice(0, 48);
          ctx.lineWidth = 3;
          ctx.strokeStyle = '#101010';
          ctx.strokeText(text, center, y);
          ctx.fillStyle = cfg.espTextColor;
          ctx.fillText(text, center, y);
          y += fontSize+3;
        };

        if (cfg.espName) {
          label(bot.name ?? bot.displayName ??
            bot.playerName ?? bot.nick ?? 'Bot');
        }

        if (cfg.espDistance) label(Math.round(entityDistance)+' m');

        if (cfg.espHealthText) {
          label(Math.max(0,Math.round(Number(bot.health)||0))+' HP');
        }

        if (cfg.espWeapon) {
          const weapon = heldWeaponInfo(bot);
          drawWeaponIcon(weapon.id, center, y);
          y += 25;
          label(weapon.name);
        }

        ctx.restore();
      }
      ctx.globalAlpha=1;
    }
  }

  const weaponIcons = new Map();
  const weaponNames = {ak47:'AK-47',m4a1:'M4A1',m4a1s:'M4A1-S',awp:'AWP',ssg08:'SSG 08',deagle:'Desert Eagle',glock:'Glock-18',usp:'USP-S',usps:'USP-S',p250:'P250',mp9:'MP9',mp7:'MP7',mac10:'MAC-10',ump45:'UMP-45',p90:'P90',famas:'FAMAS',galil:'Galil AR',galilar:'Galil AR',aug:'AUG',sg553:'SG 553',scar20:'SCAR-20',g3sg1:'G3SG1',xm1014:'XM1014',mag7:'MAG-7',sawedoff:'Sawed-Off',nova:'Nova',negev:'Negev',m249:'M249',tec9:'Tec-9',fiveseven:'Five-SeveN',elite:'Dual Berettas',hkp2000:'P2000',cz75a:'CZ75-Auto',revolver:'R8 Revolver',he:'HE Grenade',flash:'Flashbang',smoke:'Smoke Grenade',molotov:'Molotov',incend:'Incendiary Grenade',decoy:'Decoy',knife:'Knife',c4:'C4',zeus:'Zeus x27'};

  function heldWeaponInfo(entity) {
    const raw = entity.weapon ?? entity.weapons?.current ?? entity.currentWeapon ??
      entity.activeWeapon ?? entity.gun ?? entity.weaponName;
    const id = norm(typeof raw === 'object' ? raw?.id ?? raw?.name : raw);
    return {id, name: weaponNames[id] || (id ? titlecase(id) : 'Unknown')};
  }

  function drawWeaponIcon(id, x, y) {
    let entry = weaponIcons.get(id);
    if (!entry && /^[a-z0-9_]+$/.test(id)) {
      const image = new Image();
      entry = {image, ready:false};
      weaponIcons.set(id, entry);
      // Use native game artwork, with a local-file fallback to the live asset host.
      const base = location.protocol === 'file:' ? 'https://www.clutcher.io/' : document.baseURI;
      const urls = ['ui/weaponicons/' + id + '.svg', 'wpn/' + id + '.svg'];
      let attempt = 0;
      image.onload = () => { entry.ready = image.naturalWidth > 0; };
      image.onerror = () => { if (++attempt < urls.length) image.src = new URL(urls[attempt], base).href; };
      image.src = new URL(urls[0], base).href;
    }
    ctx.save();
    if (entry?.ready) {
      const scale = Math.min(58 / entry.image.naturalWidth, 21 / entry.image.naturalHeight);
      const width = entry.image.naturalWidth * scale, height = entry.image.naturalHeight * scale;
      ctx.drawImage(entry.image, x - width / 2, y + (22 - height) / 2, width, height);
    } else {
      // Keep an icon visible while artwork loads or when an asset is unavailable.
      ctx.translate(x - 26, y + 4);
      ctx.fillStyle = cfg.accent;
      const pistol = /glock|usp|p250|deagle|tec9|five|elite|p2000|cz75|revolver/.test(id);
      const path = isKnife(id) ? 'M3 13 L34 0 L48 1 L20 15 L15 12 L8 19 Z' :
        pistol ? 'M10 0 H43 V7 H31 L27 18 H18 L20 7 H10 Z' :
        'M0 5 H13 V2 H35 V4 H52 V7 H35 L30 11 L32 18 H26 L23 10 H16 L10 15 H0 Z';
      ctx.fill(new Path2D(path));
    }
    ctx.restore();
  }

  function initESP() {
    overlay = document.createElement('canvas');
    overlay.style.cssText =
      'position:fixed;pointer-events:none;z-index:2147483645;display:none;';

    document.documentElement.appendChild(overlay);
    ctx = overlay.getContext('2d');
    requestAnimationFrame(espFrame);
  }

  // =====================================================
  // GUI
  // =====================================================

  const svg = d => '<svg viewBox="0 0 24 24" aria-hidden="true">' + d + '</svg>';

  const ICONS = {
    Legit: svg('<circle cx="12" cy="12" r="9"/><circle cx="12" cy="12" r="4"/><path d="M12 1v4M12 19v4M1 12h4M19 12h4"/>'),
    Rage: svg('<path d="M13 3v7h6l-8 11v-7H5z"/>'),
    Visuals: svg('<circle cx="12" cy="12" r="2"/><path d="M22 12c-2.7 4.7-6 7-10 7s-7.3-2.3-10-7c2.7-4.7 6-7 10-7s7.3 2.3 10 7"/>'),
    Movement: svg('<path d="M4 17l5-5 4 3 7-8"/><path d="M15 7h5v5"/>'),
    Misc: svg('<path d="M6 4v4M6 12v8M12 4v10M12 18v2M18 4v1M18 9v11"/><circle cx="6" cy="10" r="2"/><circle cx="12" cy="16" r="2"/><circle cx="18" cy="7" r="2"/>'),
    Skins: svg('<path d="M4 4h10l6 6-10 10-6-6z"/><circle cx="9" cy="9" r="1.5"/>'),
    Settings: svg('<circle cx="12" cy="12" r="3"/><path d="M12 2v3M12 19v3M2 12h3M19 12h3M4.9 4.9L7 7M17 17l2.1 2.1M19.1 4.9L17 7M7 17l-2.1 2.1"/>')
  };

  const ACCENTS = ['#ff3b4a', '#35e0b0', '#8b7bff', '#3d8bff', '#ffc25c'];

  const elem = (tag, className, text) => {
    const x = document.createElement(tag);
    if (className) x.className = className;
    if (text != null) x.textContent = text;
    return x;
  };

  // Keep collapse state while switching tabs or rebuilding the menu.
  const collapsedModules = new Set();

  function section(name, description) {
    const panel = elem('section', 'section collapsible');
    const id = cfg.tab + ':' + name;
    panel.__bindPrimary = null;
    const heading = elem('strong', 'section-title', name);
    heading.setAttribute('role', 'button');
    heading.setAttribute('tabindex', '0');
    const sync = () => {
      const collapsed = collapsedModules.has(id);
      panel.classList.toggle('collapsed', collapsed);
      heading.setAttribute('aria-expanded', String(!collapsed));
      heading.setAttribute('aria-label', name + (collapsed ? ', expand' : ', collapse'));
    };
    const toggle = () => {
      if (collapsedModules.has(id)) collapsedModules.delete(id);
      else collapsedModules.add(id);
      sync();
    };
    panel.append(heading);
    if (description) panel.append(elem('p', 'desc', description));
    // Clicking the title, description, or unused card space toggles it.
    // Leave all actual settings and their label rows fully interactive.
    panel.addEventListener('click', event => {
      if (event.target.closest('button,input,select,textarea,label,a,[contenteditable="true"]')) return;
      toggle();
    });
    heading.addEventListener('keydown', event => {
      if (event.key === 'Enter' || event.key === ' ') {
        event.preventDefault();
        toggle();
      }
    });
    sync();
    (pageBody || page).append(panel);
    return panel;
  }

  function check(parent, label, key) {
    const row = elem('label', 'field');
    row.append(document.createTextNode(label));

    const input = elem('input');
    input.type = 'checkbox';
    input.checked = !!cfg[key];
    input.dataset.settingKey = key;

    input.onchange = () => {
      cfg[key] = input.checked;
      save();
    };

    row.append(input);
    // Only the primary control of a module receives a header bind.
    // Secondary controls (arms, weapon, fog, dropdown settings, etc.)
    // never get individual bind buttons.
    if (bindable.has(key) && parent.classList?.contains('section') && !parent.__bindPrimary) {
      parent.__bindPrimary = key;
      const heading = parent.querySelector('.section-title');
      if (heading) {
        const group = elem('span', 'bind-title-actions');
        group.append(bindButton(key, true));
        heading.append(group);
      }
    }
    parent.append(row);
    refreshBindButtons();
  }

  function aaSelect(parent, label, key, options) {
    const row = elem('label', 'field');
    row.append(elem('span', '', label));
    const input = elem('select');
    for (const [value, text] of options) {
      const option = elem('option', '', text);
      option.value = value;
      input.append(option);
    }
    input.value = cfg[key];
    input.onchange = () => { cfg[key] = input.value; save(); };
    row.append(input);
    parent.append(row);
  }

  function aaRange(parent, label, key, min, max, step, unit) {
    const row = elem('label', 'field');
    const text = elem('span');
    const input = elem('input');
    Object.assign(input, {type:'range', min:String(min), max:String(max), step:String(step), value:String(cfg[key])});
    input.style.width = '150px';
    const update = () => {
      cfg[key] = aaNumber(input.value, min, max, defaults[key]);
      text.textContent = label + ': ' + cfg[key] + unit;
    };
    update();
    input.oninput = () => { update(); save(); };
    row.append(text, input);
    parent.append(row);
  }

  function colorField(parent, label, key) {
    const row = elem('label', 'field');
    row.append(elem('span', '', label));
    const input = elem('input');
    input.type = 'color';
    input.value = cfg[key];
    input.oninput = () => { cfg[key] = input.value; save(); };
    row.append(input);
    parent.append(row);
  }

  function textField(parent,label,key,placeholder='') {
    const row=elem('label','field');
    row.append(elem('span','',label));
    const input=elem('input');
    input.type='text';input.value=cfg[key]||'';input.placeholder=placeholder;
    input.onchange=()=>{cfg[key]=input.value.trim();save();};
    row.append(input);parent.append(row);
  }

  function renderNav() {
    nav.replaceChildren();

    tabs.forEach(name => {
      const b = elem('button', 'nav' + (cfg.tab === name ? ' active' : ''));
      b.innerHTML = ICONS[name] + '<span></span>';
      b.lastChild.textContent = name;

      b.onclick = () => {
        cfg.tab = name;
        save();
        renderNav();
        renderPage();
      };

      nav.append(b);
    });
  }

  function renderPage() {
    page.replaceChildren();
    pageBody = elem('div', 'panel-grid');
    page.append(elem('h2', '', cfg.tab), pageBody);

    if (cfg.tab === 'Rage') {
      const aa = section('Anti Aim', 'Third-person model angles.');
      check(aa, 'Enable Anti Aim', 'antiAim');
      aaSelect(aa, 'Yaw base', 'aaBase', [['target','At targets'],['view','View direction']]);
      aaSelect(aa, 'Yaw mode', 'aaYawMode', [['backward','Backward'],['forward','Forward'],['spin','Spin']]);
      aaRange(aa, 'Yaw offset', 'aaYawOffset', -180, 180, 1, ' deg');
      aaSelect(aa, 'Pitch', 'aaPitch', [['down','Down'],['up','Up'],['level','Level'],['view','View pitch']]);
      aaRange(aa, 'Spin speed', 'aaSpinSpeed', -360, 360, 5, ' deg/s');
      aaSelect(aa, 'Jitter', 'aaJitter', [['off','Off'],['center','Center'],['offset','Offset'],['three','Three-way'],['random','Random']]);
      aaRange(aa, 'Jitter range', 'aaJitterRange', 0, 180, 1, ' deg');
      aaRange(aa, 'Angle interval', 'aaInterval', 40, 1000, 10, ' ms');
      aaSelect(aa, 'Simulated desync', 'aaDesync', [['off','Off'],['static','Static'],['alternate','Alternating'],['sway','Sway']]);
      aaRange(aa, 'Body offset', 'aaDesyncAmount', 0, 60, 1, ' deg');
      check(aa, 'Invert body offset', 'aaInvert');

      const network = section('Fake Lag / Freeze',
        'Hold your reported multiplayer position while local movement stays active.');
      check(network, 'Enable Fake Lag', 'fakeLag');
      const lagRow = elem('label', 'field');
      const lagLabel = elem('span', '', `Lag interval: ${cfg.fakeLagLimit} ms`);
      const lagSlider = elem('input');
      Object.assign(lagSlider, { type: 'range', min: '0', max: '400', step: '5', value: String(cfg.fakeLagLimit) });
      lagSlider.style.width = '170px';
      lagSlider.oninput = () => {
        cfg.fakeLagLimit = Number(lagSlider.value);
        lagLabel.textContent = `Lag interval: ${cfg.fakeLagLimit} ms`;
        save();
      };
      lagRow.append(lagLabel, lagSlider);
      network.append(lagRow);
      const modeRow = elem('label', 'field');
      modeRow.append(elem('span', '', 'Lag mode'));
      const lagMode = elem('select');
      for (const [value, label] of [['always', 'Always'], ['move', 'On move'], ['random', 'Random']]) {
        const option = elem('option', '', label);
        option.value = value;
        lagMode.append(option);
      }
      lagMode.value = cfg.fakeLagMode;
      lagMode.onchange = () => { cfg.fakeLagMode = lagMode.value; save(); };
      modeRow.append(lagMode);
      network.append(modeRow);
      check(network, 'Enable Fake Freeze', 'fakeFreeze');

      const rage = section(
        'Rage Bot',
        'Automatically target enemies using damageable collision hitboxes.'
      );
      check(rage, 'Enable Rage Bot', 'rageBot');
      check(rage, 'Hitbox resolver', 'rageResolver');
      check(rage, 'Full-map wall penetration', 'rageFullMapWallbang');
      check(rage, 'Auto Crouch on Target', 'rageAutoCrouch');
      check(rage, 'Auto Stop on Target', 'rageAutoStop');
      aaSelect(rage, 'Target priority', 'ragePriority', [['crosshair','Crosshair'],['distance','Distance'],['health','Lowest health']]);
      check(rage, 'Force Head Targeting', 'rageForceHead');
      check(rage, 'Rage Damage Boost', 'rageDamageBoost');
      aaRange(rage, 'Damage scale', 'rageDamagePercent', 10, 500, 10, '%');
      check(rage, 'Sticky Rage Target', 'rageTargetLock');
      check(rage, 'Body Aim on Low HP', 'rageBodyAimLowHp');
      aaRange(rage, 'Body aim health', 'rageBodyHp', 1, 100, 1, ' HP');
      check(rage, 'Auto Jump on Target', 'rageAutoJump');
      // Multi-select target area: independent checkboxes inside a dropdown.
      const area = elem('div', 'rage-area-picker');
      const areaButton = elem('button', 'action', 'Target Area v');
      areaButton.type = 'button';
      areaButton.setAttribute('aria-expanded', 'false');
      const areaOptions = elem('div', 'rage-area-options');
      areaOptions.hidden = true;
      for (const [key, label] of [
        ['rageAreaHead','Head'], ['rageAreaNeckChest','Neck/Chest'],
        ['rageAreaArms','Arms'], ['rageAreaTorso','Torso'], ['rageAreaLegs','Legs']
      ]) {
        const row = elem('label', 'field');
        row.append(document.createTextNode(label));
        const input = elem('input');
        input.type = 'checkbox';
        input.checked = cfg[key];
        input.onchange = () => { cfg[key] = input.checked; save(); };
        row.append(input);
        areaOptions.append(row);
      }
      areaButton.onclick = () => {
        areaOptions.hidden = !areaOptions.hidden;
        areaButton.setAttribute('aria-expanded', String(!areaOptions.hidden));
      };
      area.append(areaButton, areaOptions);
      rage.append(area);
      check(rage, 'Auto fire', 'rageAutoFire');

      const rageRow = elem('label', 'field');
      const rageLabel = elem('span', '', `Aim FOV: ${cfg.rageFov} deg`);
      const rageSlider = elem('input');
      rageSlider.type = 'range';
      rageSlider.min = '1';
      rageSlider.max = '180';
      rageSlider.step = '1';
      rageSlider.value = String(cfg.rageFov);
      rageSlider.style.width = '170px';
      rageSlider.oninput = () => {
        cfg.rageFov = Number(rageSlider.value);
        rageLabel.textContent = `Aim FOV: ${cfg.rageFov} deg`;
        save();
      };
      rageRow.append(rageLabel, rageSlider);
      rage.append(rageRow);

      const rapid = section(
        'Rapid Fire',
        'Remove the delay between shots while firing; semi-auto weapons keep firing while held.'
      );
      check(rapid, 'Enable Rapid Fire', 'rapidFire');

      const combat = section('Weapon Modifiers', 'Direct local weapon behavior controls.');
      check(combat, 'No Spread', 'noSpread');
      check(combat, 'No Recoil', 'noRecoil');
      check(combat, 'Infinite Ammo', 'infiniteAmmo');
      check(combat, 'Lethal Shots', 'lethalShots');

    } else if (cfg.tab === 'Legit') {
      const s = section(
        'Silent Aim',
        'Redirect locally fired bullets toward enemy head bones within the crosshair FOV without moving your camera.'
      );

      check(s, 'Enable Silent Aim', 'silentAim');
      check(s, 'Show FOV circle', 'silentCircle');

      const row = elem('label', 'field');
      const label = elem('span', '', `FOV radius: ${cfg.silentFov} px`);
      const slider = elem('input');

      slider.type = 'range';
      slider.min = '20';
      slider.max = '500';
      slider.step = '5';
      slider.value = String(cfg.silentFov);
      slider.style.width = '170px';

      slider.oninput = () => {
        cfg.silentFov = Number(slider.value);
        label.textContent = `FOV radius: ${cfg.silentFov} px`;
        save();
      };

      row.append(label, slider);
      s.append(row);
      colorField(s, 'FOV circle color', 'silentCircleColor');
      aaRange(s, 'FOV circle opacity', 'silentCircleOpacity', 10, 100, 1, '%');
      aaRange(s, 'FOV circle width', 'silentCircleWidth', .5, 5, .25, ' px');

      const legitAim = section(
        'Legit Aimbot',
        'Smooth visible aiming with configurable targeting and speed limits.'
      );
      check(legitAim, 'Enable Legit Aimbot', 'legitAimbot');
      aaSelect(legitAim, 'Activation', 'legitAimMode', [['fire','While firing'],['always','Always']]);
      aaSelect(legitAim, 'Target bone', 'legitAimBone', [['head','Head'],['chest','Chest'],['nearest','Nearest core bone']]);
      aaRange(legitAim, 'Aim FOV', 'legitAimFov', .5, 30, .5, ' deg');
      aaRange(legitAim, 'Smoothing', 'legitSmooth', 1, 30, .5, '');
      aaRange(legitAim, 'Max aim speed', 'legitMaxSpeed', 30, 720, 10, ' deg/s');
      aaRange(legitAim, 'Max distance', 'legitAimMaxDistance', 10, 300, 5, ' m');
      check(legitAim, 'Check smoke visibility', 'legitCheckSmoke');
      check(legitAim, 'Aim Deadzone', 'legitDeadzone');
      aaRange(legitAim, 'Deadzone', 'legitDeadzoneFov', 0, 5, .1, ' deg');
      check(legitAim, 'Sticky Target Lock', 'legitTargetLock');
      check(legitAim, 'Humanized Aim Drift', 'legitHumanize');
      aaRange(legitAim, 'Humanize amount', 'legitHumanizeAmount', 0, 2, .05, ' deg');
      check(legitAim, 'Show Legit FOV Circle', 'legitAimCircle');
      check(legitAim, 'Dynamic Smoothing', 'legitDynamicSmooth');
      check(legitAim, 'Aim Reaction Delay', 'legitReactionDelay');
      aaRange(legitAim, 'Reaction time', 'legitReactionMs', 0, 1000, 10, ' ms');

      const legitHelpers=section('Legit Helpers','Trigger, recoil and reload assistance.');
      check(legitHelpers, 'Legit Recoil Control', 'legitRcs');
      aaRange(legitHelpers, 'RCS amount', 'legitRcsPercent', 0, 100, 5, '%');
      check(legitHelpers, 'Triggerbot', 'triggerBot');
      aaRange(legitHelpers, 'Trigger radius', 'triggerRadius', 2, 40, 1, ' px');
      check(legitHelpers, 'Trigger Delay', 'triggerDelay');
      aaRange(legitHelpers, 'Trigger delay', 'triggerDelayMs', 0, 500, 10, ' ms');
      check(legitHelpers, 'Auto Reload', 'autoReload');
      aaRange(legitHelpers, 'Reload at ammo', 'autoReloadThreshold', 0, 15, 1, '');

    } else if (cfg.tab === 'Movement') {
      const bhop = section(
        'Auto Bhop',
        'Automatically jump again when you land while holding Space.'
      );
      check(bhop, 'Enable Auto Bhop', 'autoBhop');

      const superBhop = section(
        'Super Bhop',
        'Automatically bunny hop while holding Space and boost airborne movement speed.'
      );
      check(superBhop, 'Enable Super Bhop', 'superBhop');
      aaRange(superBhop, 'Air speed', 'superBhopSpeedPercent', 100, 500, 5, '%');

      const strafe = section(
        'Auto Strafe',
        'Air strafe with WASD, mouse movement, or Rage mode.'
      );
      check(strafe, 'Enable Auto Strafe', 'autoStrafe');

      const strafeModes = elem('div', 'modes');
      [['wasd', 'WASD'], ['legit', 'Legit'], ['rage', 'Rage']].forEach(([id, label]) => {
        const button = elem('button',
          'mode' + (cfg.strafeMode === id ? ' selected' : ''), label);
        button.onclick = () => {
          cfg.strafeMode = id;
          save();
          renderPage();
        };
        strafeModes.append(button);
      });
      strafe.append(strafeModes);

      if (cfg.strafeMode === 'wasd' || cfg.strafeMode === 'rage') {
        const rageMode = cfg.strafeMode === 'rage';
        const speedKey = rageMode ? 'rageSpeedPercent' : 'strafeSpeedPercent';
        // Reuse the existing switch, field, number, and slider styling.
        if (!rageMode) check(strafe, 'Custom WASD speed limit', 'strafeCustomSpeed');

        const speedRow = elem('div', 'field');
        const speedLabel = elem('span', '', 'Speed limit: 100.00%');
        const speedControls = elem('div');
        speedControls.style.cssText = 'display:flex;align-items:center;gap:10px;';

        const speedSlider = elem('input');
        speedSlider.type = 'range';
        speedSlider.min = '50';
        speedSlider.max = '150';
        speedSlider.step = '0.01';
        speedSlider.style.width = '160px';
        speedSlider.setAttribute('aria-label', rageMode ? 'Rage speed percent' : 'WASD speed limit percent');

        const speedNumber = elem('input');
        speedNumber.type = 'number';
        speedNumber.min = '50';
        speedNumber.max = '150';
        speedNumber.step = '0.01';
        speedNumber.style.width = '85px';
        speedNumber.setAttribute('aria-label', rageMode ? 'Exact Rage speed percent' : 'Exact WASD speed limit percent');

        const speedInitial = Number(cfg[speedKey]);
        cfg[speedKey] = Number.isFinite(speedInitial)
          ? Math.round(Math.max(50, Math.min(150, speedInitial)) * 100) / 100
          : 100;

        const refreshSpeed = () => {
          const text = cfg[speedKey].toFixed(2);
          speedLabel.textContent = `${rageMode ? 'Rage speed' : 'Speed limit'}: ${text}%`;
          speedSlider.value = String(cfg[speedKey]);
          speedNumber.value = text;
        };
        const changeSpeed = value => {
          if (value === '' || !Number.isFinite(Number(value))) return;
          cfg[speedKey] = Math.round(
            Math.max(50, Math.min(150, Number(value))) * 100
          ) / 100;
          refreshSpeed();
          save();
        };
        speedSlider.oninput = () => changeSpeed(speedSlider.value);
        speedNumber.onchange = () => changeSpeed(speedNumber.value);
        speedControls.append(speedSlider, speedNumber);
        speedRow.append(speedLabel, speedControls);
        strafe.append(speedRow);
        refreshSpeed();
      }

      const movementMods = section('Movement Modifiers', 'Tune grounded and airborne movement.');
      check(movementMods, 'Ground Speed Boost', 'groundSpeed');
      aaRange(movementMods, 'Ground speed', 'groundSpeedPercent', 100, 300, 5, '%');
      check(movementMods, 'Crouch Speed Boost', 'crouchBoost');
      aaRange(movementMods, 'Crouch speed', 'crouchSpeedPercent', 100, 300, 5, '%');
      check(movementMods, 'Jump Boost', 'jumpBoost');
      aaRange(movementMods, 'Jump power', 'jumpBoostPercent', 100, 300, 5, '%');
      check(movementMods, 'Low Gravity', 'lowGravity');
      aaRange(movementMods, 'Gravity', 'gravityPercent', 0, 100, 5, '%');
      check(movementMods, 'Auto Walk', 'autoWalk');
      check(movementMods, 'Air Brake', 'airBrake');
      aaRange(movementMods, 'Brake strength', 'airBrakeStrength', 10, 100, 5, '%');
      check(movementMods, 'Edge Jump', 'edgeJump');
      check(movementMods, 'Preserve Landing Momentum', 'momentumPreserve');
      check(movementMods, 'Ladder Speed Boost', 'ladderBoost');
      aaRange(movementMods, 'Ladder speed', 'ladderSpeedPercent', 100, 300, 5, '%');
      check(movementMods, 'Air Acceleration Boost', 'airAccelBoost');
      aaRange(movementMods, 'Air acceleration', 'airAccelPercent', 100, 300, 5, '%');
      check(movementMods, 'Auto Crouch in Air', 'autoAirCrouch');
      check(movementMods, 'Fall Speed Limiter', 'fallSpeedLimiter');
      aaRange(movementMods, 'Maximum fall speed', 'maxFallSpeed', 2, 30, .5, ' m/s');
      check(movementMods, 'Reverse Movement', 'reverseMovement');
      check(movementMods, 'Auto Crouch on Landing', 'autoLandCrouch');
      aaRange(movementMods, 'Landing crouch time', 'landCrouchTime', 50, 1000, 25, ' ms');


    } else if (cfg.tab === 'Visuals') {
      const chams = section('Viewmodel Chams', 'Only your local first-person arms and held weapon. Keeps existing client skin textures.');
      check(chams, 'Enable Viewmodel Chams', 'vmChams');
      check(chams, 'Apply to arms / gloves', 'vmArms');
      aaRange(chams, 'Arm opacity', 'vmArmOpacity', 0, 100, 1, '%');
      aaRange(chams, 'Arm brightness', 'vmArmBrightness', 0, 300, 1, '%');
      check(chams, 'Apply to held weapon', 'vmWeapon');
      aaRange(chams, 'Weapon opacity', 'vmWeaponOpacity', 0, 100, 1, '%');
      aaRange(chams, 'Tint strength', 'vmTintStrength', 0, 100, 1, '%');
      const tintRow = elem('label', 'field');
      tintRow.append(elem('span', '', 'Tint color'));
      const tintInput = elem('input');
      tintInput.type = 'color'; tintInput.value = cfg.vmTint;
      tintInput.oninput = () => { cfg.vmTint = tintInput.value; save(); };
      tintRow.append(tintInput); chams.append(tintRow);

      const viewmodel = section('Viewmodel Positioning', 'Customize weapon placement and motion without changing native settings.');
      check(viewmodel, 'Enable Custom Viewmodel', 'vmCustom');
      aaRange(viewmodel, 'Viewmodel FOV', 'vmFov', 54, 80, 1, ' deg');
      aaRange(viewmodel, 'Horizontal offset', 'vmOffsetX', -3, 3, .05, '');
      aaRange(viewmodel, 'Depth offset', 'vmOffsetY', -3, 3, .05, '');
      aaRange(viewmodel, 'Vertical offset', 'vmOffsetZ', -3, 3, .05, '');
      aaRange(viewmodel, 'Pitch', 'vmPitch', -30, 30, 1, ' deg');
      aaRange(viewmodel, 'Yaw', 'vmYaw', -30, 30, 1, ' deg');
      aaRange(viewmodel, 'Roll', 'vmRoll', -45, 45, 1, ' deg');
      aaRange(viewmodel, 'Bob amount', 'vmBobPercent', 0, 200, 5, '%');
      aaRange(viewmodel, 'Sway amount', 'vmSwayPercent', 0, 200, 5, '%');

      const world = section('World Settings', 'Client-side atmospheric effects. Does not affect other players.');
      check(world, 'Enable World Settings', 'worldEnabled');
      const modeRow = elem('label', 'field');
      const modeLabel = elem('span', '', 'Time of day');
      const mode = elem('select');
      for (const [value, label] of [['day','Day'], ['night','Night']]) {
        const option = elem('option', '', label);
        option.value = value;
        mode.append(option);
      }
      mode.value = cfg.worldMode;
      mode.onchange = () => { cfg.worldMode = mode.value; save(); };
      modeRow.append(modeLabel, mode);
      world.append(modeRow);
      aaRange(world, 'Night strength', 'worldNightStrength', 0, 100, 1, '%');
      check(world, 'Custom fog', 'worldFog');
      const fogModeRow = elem('label', 'field');
      fogModeRow.append(elem('span', '', 'Fog type'));
      const fogModeSelect = elem('select');
      for (const [value, label] of [['distance','Distance (linear)'],['exponential','Atmospheric (exponential)']]) {
        const option = elem('option', '', label); option.value = value; fogModeSelect.append(option);
      }
      fogModeSelect.value = cfg.worldFogMode;
      fogModeSelect.onchange = () => { cfg.worldFogMode = fogModeSelect.value; save(); renderPage(); };
      fogModeRow.append(fogModeSelect); world.append(fogModeRow);
      if (cfg.worldFogMode === 'distance') {
        aaRange(world, 'Fog starts', 'worldFogNear', 0, 250, 1, ' m');
        aaRange(world, 'Fog fully opaque', 'worldFogFar', 10, 500, 1, ' m');
      } else {
        aaRange(world, 'Fog density', 'worldFogDensity', 0, 100, 1, '%');
      }
      const fogRow = elem('label', 'field');
      fogRow.append(elem('span', '', 'Fog color'));
      const fogInput = elem('input');
      fogInput.type = 'color'; fogInput.value = cfg.worldFogColor;
      fogInput.oninput = () => { cfg.worldFogColor = fogInput.value; save(); };
      fogRow.append(fogInput); world.append(fogRow);
      // Weather belongs inside World Settings rather than a separate module card.
      world.append(elem('div', 'world-subheading', 'Weather / Rain'));
      world.append(elem('p', 'desc', 'Animated 3D rain that works alongside Day/Night and fog.'));
      check(world, 'Enable Rain', 'worldRain');
      aaRange(world, 'Rain intensity', 'worldRainIntensity', 0, 100, 1, '%');
      aaRange(world, 'Rain fall speed', 'worldRainSpeed', 10, 100, 1, '%');
      aaRange(world, 'Rain opacity', 'worldRainOpacity', 10, 100, 1, '%');
      colorField(world, 'Rain color', 'worldRainColor');

      const sceneFx=section('Scene Effects','Custom sky and screen post-processing controls.');
      check(sceneFx,'Custom Skybox','customSkybox');
      textField(sceneFx,'Skybox image URL','customSkyboxUrl','https://.../sky.jpg');
      check(sceneFx,'Screen Vignette','screenVignette');
      aaRange(sceneFx,'Vignette opacity','vignetteOpacity',0,100,5,'%');
      colorField(sceneFx,'Vignette color','vignetteColor');
      check(sceneFx,'World Saturation','worldSaturation');
      aaRange(sceneFx,'Saturation','saturationPercent',0,300,5,'%');

      const s = section(
        'Player ESP',
        'Configurable player overlays using the animated game model.'
      );

      check(s, 'Enable Bone ESP', 'boneESP');
      check(s, 'Box ESP', 'boxESP');
      check(s, 'Corner Box ESP', 'cornerBoxESP');
      check(s, 'Filled Boxes', 'boxFill');
      aaRange(s, 'Fill opacity', 'boxFillOpacity', 1, 50, 1, '%');
      check(s, 'Head Dot ESP', 'headDotESP');
      aaRange(s, 'Head dot size', 'headDotSize', 1, 8, .5, ' px');
      check(s, 'Rainbow Enemy ESP', 'rainbowESP');
      aaRange(s, 'Rainbow speed', 'rainbowSpeed', 10, 200, 5, '%');

      const cols = elem('div', 'cols');
      check(cols, 'Show teammates', 'boneESPTeam');
      check(cols, 'Health bar (left)', 'espHealth');
      check(cols, 'Health text', 'espHealthText');
      check(cols, 'Armor bar (right)', 'espArmor');
      check(cols, 'Player name (below)', 'espName');
      check(cols, 'Distance (below)', 'espDistance');
      check(cols, 'Held weapon icon + name', 'espWeapon');
      s.append(cols);
      check(s, 'Bottom tracers', 'espTracers');
      aaRange(s, 'Skeleton width', 'espLineWidth', .5, 6, .25, ' px');
      aaRange(s, 'Box thickness', 'boxThickness', .5, 6, .25, ' px');
      aaRange(s, 'Joint dot size', 'espJointSize', 0, 6, .25, ' px');
      aaRange(s, 'Maximum distance', 'espMaxDistance', 0, 300, 5, ' m');
      aaRange(s, 'Global opacity', 'espOpacity', 10, 100, 5, '%');
      aaRange(s, 'Text size', 'espFontSize', 8, 24, 1, ' px');
      colorField(s, 'Enemy color', 'espEnemyColor');
      colorField(s, 'Teammate color', 'espTeamColor');
      colorField(s, 'Text color', 'espTextColor');
      colorField(s, 'Health bar color', 'espHealthColor');
      colorField(s, 'Armor bar color', 'espArmorColor');
      colorField(s, 'Helmet armor color', 'espHelmetColor');


      const arrows = section(
        'Arrow ESP',
        'Triangles around the crosshair show the direction of every living enemy, including behind you.'
      );
      check(arrows, 'Enable Arrow ESP', 'arrowESP');
      check(arrows, 'Show Arrow Distance', 'arrowDistance');
      aaRange(arrows, 'Ring radius', 'arrowRadiusPercent', 25, 90, 1, '%');
      aaRange(arrows, 'Arrow size', 'arrowSize', 8, 30, 1, ' px');
      colorField(arrows, 'Visible enemy color', 'arrowVisibleColor');
      colorField(arrows, 'Hidden enemy color', 'arrowHiddenColor');

      const crosshair = section(
        'Crosshair',
        'A clean overlay crosshair independent of the game HUD.'
      );
      check(crosshair, 'Enable Crosshair', 'crosshairEnabled');
      check(crosshair, 'Center Dot', 'crosshairDot');
      aaRange(crosshair, 'Dot size', 'crosshairDotSize', 1, 6, .5, ' px');
      check(crosshair, 'Crosshair Outline', 'crosshairOutline');
      aaRange(crosshair, 'Line size', 'crosshairSize', 2, 30, 1, ' px');
      aaRange(crosshair, 'Center gap', 'crosshairGap', 0, 20, 1, ' px');
      aaRange(crosshair, 'Line width', 'crosshairWidth', 1, 6, .5, ' px');
      colorField(crosshair, 'Crosshair color', 'crosshairColor');

      const hud = section('HUD / Camera', 'Local feedback and camera controls.');
      check(hud, 'Force Hitmarkers', 'forceHitmarkers');
      check(hud, 'Force Bullet Tracers', 'localTracers');
      check(hud, 'AMNESIS Watermark', 'watermark');
      check(hud, 'Velocity HUD', 'velocityHud');
      check(hud, 'Active Keybind List', 'keybindList');
      check(hud, 'Override Camera FOV', 'fovOverride');
      aaRange(hud, 'Camera FOV', 'cameraFov', 60, 130, 1, ' deg');

      const radar=section('Radar ESP','Player-relative radar independent of the native HUD.');
      check(radar, 'Enable Radar ESP', 'radarESP');
      aaRange(radar, 'Radar range', 'radarRange', 10, 200, 5, ' m');
      aaRange(radar, 'Radar size', 'radarSize', 80, 220, 5, ' px');

      const third = section(
        'Third Person',
        'Follow your character from behind instead of using the first-person camera.'
      );
      check(third, 'Enable Third Person', 'thirdPerson');

      // Reuse the menu's field, slider, and number-input styling.
      // The slider updates the chase camera immediately, no reload required.
      const distanceRow = elem('label', 'field');
      const initialDistance = Number(cfg.thirdDistance);
      cfg.thirdDistance = Number.isFinite(initialDistance)
        ? Math.max(0.02, Math.min(5, initialDistance))
        : 3.30;
      const distanceLabel = elem('span', '',
        `Camera distance: ${cfg.thirdDistance.toFixed(2)}`);
      const distanceSlider = elem('input');
      distanceSlider.type = 'range';
      distanceSlider.min = '0.02';
      distanceSlider.max = '5.00';
      distanceSlider.step = '0.01';
      distanceSlider.value = String(cfg.thirdDistance);
      distanceSlider.style.width = '170px';

      const exactRow = elem('label', 'field');
      exactRow.append(document.createTextNode('Precise distance'));
      const distanceNumber = elem('input');
      distanceNumber.type = 'number';
      distanceNumber.min = '0.02';
      distanceNumber.max = '5.00';
      distanceNumber.step = '0.01';
      distanceNumber.value = cfg.thirdDistance.toFixed(2);

      function setThirdDistance(value, fromSlider) {
        if (!Number.isFinite(value)) return;
        const rounded = Math.round(Math.max(0.02, Math.min(5, value)) * 100) / 100;
        cfg.thirdDistance = rounded;
        distanceLabel.textContent = `Camera distance: ${rounded.toFixed(2)}`;
        distanceSlider.value = String(rounded);
        if (fromSlider) distanceNumber.value = rounded.toFixed(2);
        save();
      }

      distanceSlider.oninput = () =>
        setThirdDistance(Number(distanceSlider.value), true);
      distanceNumber.oninput = () => {
        if (distanceNumber.value !== '') {
          setThirdDistance(Number(distanceNumber.value), false);
        }
      };
      distanceNumber.onblur = () => {
        distanceNumber.value = cfg.thirdDistance.toFixed(2);
      };
      distanceRow.append(distanceLabel, distanceSlider);
      exactRow.append(distanceNumber);
      third.append(distanceRow, exactRow);

    } else if (cfg.tab === 'Misc') {
      hitsoundControls();

      const utility = section('Utility', 'World and first-person quality-of-life controls.');
      check(utility, 'Fullbright', 'fullbright');
      aaRange(utility, 'Fullbright strength', 'fullbrightStrength', 100, 250, 5, '%');
      check(utility, 'Remove Flash', 'removeFlash');
      check(utility, 'Remove Smoke', 'noSmoke');
      check(utility, 'Hide Scope Overlay', 'noScopeOverlay');
      check(utility, 'Hide Viewmodel', 'hideViewmodel');

      const session=section('Session / Interface','Local session statistics and presentation controls.');
      check(session, 'Session Kill Counter', 'killCounter');
      check(session, 'Session Damage Counter', 'damageCounter');
      check(session, 'Hide Native HUD', 'hideNativeHud');
      check(session, 'Remove Blood Effects', 'noBlood');
      check(session, 'Auto Inspect After Kill', 'autoInspectKill');

      const info=section('Information HUD','Optional local utility readouts.');
      check(info,'Clock HUD','clockHud');
      check(info,'FPS HUD','fpsHud');
      check(info,'Coordinates HUD','coordinatesHud');

    } else if (cfg.tab === 'Skins') {
      const changer=section('Runtime Skin Changer',
        'Local cosmetic overrides only. Native inventory data is never modified.');
      check(changer,'Enable Skin Changer','skinChanger');
      check(changer,'Random Knife After Kill','skinRandomKnife');
      check(changer,'Random Gloves After Kill','skinRandomGloves');
      check(changer,'Bare Hands','skinBareHands');
      aaSelect(changer,'Held weapon finish','skinCurrentMode',[
        ['native','Native equipped'],['default','Default finish']
      ]);

      const knives=section('Knife Model','Choose a local first and third-person knife model.');
      aaSelect(knives,'Knife model','skinKnifeModel',[
        ['native','Native equipped'],['default','Team default'],
        ...SKIN_KNIFE_MODELS.map(id=>[id,titlecase(id.replace(/^knife_/,''))])
      ]);

      const gloves=section('Glove Model','Choose an unpainted local glove model.');
      aaSelect(gloves,'Glove model','skinGloveModel',[
        ['native','Native equipped'],['default','Team default'],
        ...SKIN_GLOVE_MODELS.map(id=>[id,titlecase(id)])
      ]);
      const resetSkins=elem('button','action','Reset Skin Overrides');
      resetSkins.onclick=()=>{
        cfg.skinCurrentMode='native';cfg.skinKnifeModel='native';
        cfg.skinGloveModel='native';cfg.skinRandomKnife=false;
        cfg.skinRandomGloves=false;cfg.skinBareHands=false;
        save();renderPage();
      };
      gloves.append(resetSkins);
      gloves.append(elem('p','note',
        'This game build currently supports knife and glove model swaps; firearm paint kits are unavailable.'));

    } else if (cfg.tab === 'Settings') {
      configControls();
      const s = section('Appearance', 'Configure the menu.');

      const compact = elem('label', 'field');
      compact.append(document.createTextNode('Compact window'));

      const cb = elem('input');
      cb.type = 'checkbox';
      cb.checked = cfg.compact;

      cb.onchange = () => {
        cfg.compact = cb.checked;
        box.classList.toggle('compact', cfg.compact);
        position();
        save();
      };

      compact.append(cb);
      s.append(compact);
      check(s,'Blurred Interface','uiBlur');
      check(s,'Interface Animations','uiAnimations');
      check(s,'Transparent Interface','uiTransparency');
      aaRange(s,'Interface opacity','uiOpacity',40,100,1,'%');

      const colorRow = elem('div', 'field');
      colorRow.append(elem('span', '', 'Accent color'));

      const dots = elem('div', 'dots');
      const picker = elem('input');
      picker.type = 'color';
      picker.value = cfg.accent;

      const setAccent = color => {
        cfg.accent = color;
        box.style.setProperty('--accent', color);
        picker.value = color;
        dots.querySelectorAll('.dot').forEach(d =>
          d.classList.toggle('on', d.dataset.c === color));
        save();
      };

      ACCENTS.forEach(color => {
        const d = elem('button', 'dot' + (cfg.accent === color ? ' on' : ''));
        d.dataset.c = color;
        d.style.background = color;
        d.title = color;
        d.onclick = () => setAccent(color);
        dots.append(d);
      });

      picker.oninput = () => setAccent(picker.value);

      dots.append(picker);
      colorRow.append(dots);
      s.append(colorRow);

      const reset = elem('button', 'action', 'Reset window position');

      reset.onclick = () => {
        cfg.x = null;
        cfg.y = null;
        position();
        save();
      };

      s.append(reset);

    } else {
      const s = elem('section', 'section');
      const empty = elem('div', 'empty');
      empty.innerHTML = ICONS[cfg.tab] || '';
      empty.append(
        elem('b', '', 'No modules yet'),
        elem('span', '', 'New options will show up here.')
      );
      s.append(empty);
      pageBody.append(s);
    }
  }

  function position() {
    const width = Math.min(1040, innerWidth - 20);
    const height = Math.min(cfg.compact ? 500 : 680, innerHeight - 20);

    cfg.x = Math.max(10, Math.min(
      cfg.x ?? (innerWidth - width) / 2,
      innerWidth - width - 10
    ));

    cfg.y = Math.max(10, Math.min(
      cfg.y ?? (innerHeight - height) / 2,
      innerHeight - height - 10
    ));

    box.style.left = cfg.x + 'px';
    box.style.top = cfg.y + 'px';
  }

  function openMenu(show) {
    if (!show && awaitingBind) { awaitingBind = null; refreshBindButtons(); }
    if (!box) return;

    uiOpen = show;
    box.classList.toggle('open', show);

    if (show) {
      lockEl = document.pointerLockElement;
      hadLock = !!lockEl;

      if (hadLock) {
        try { document.exitPointerLock(); } catch {}
      }

      document.body.style.cursor = 'default';
    } else {
      document.body.style.cursor = '';

      if (hadLock) {
        const target = lockEl?.isConnected
          ? lockEl
          : document.querySelector('#game canvas');

        if (target && !document.pointerLockElement) {
          try {
            Promise.resolve(target.requestPointerLock()).catch(() => {});
          } catch {}
        }
      }

      hadLock = false;
      lockEl = null;
    }
  }

  const focused = () =>
    uiOpen && root?.activeElement?.matches?.(
      'input,textarea,select,[contenteditable="true"]'
    );

  for (const type of ['keydown', 'keyup', 'keypress']) {
    window.addEventListener(type, e => {
      if (focused()) e.stopImmediatePropagation();
    }, true);
  }

  for (const type of ['keydown', 'keyup']) {
    window.addEventListener(type, e => {
      if (e.code !== 'ShiftRight') return;

      e.preventDefault();
      e.stopImmediatePropagation();

      if (type === 'keydown' && !e.repeat && !focused()) {
        openMenu(!uiOpen);
      }
    }, true);
  }

  function initialize() {
    if (!document.body || document.getElementById('amnesis-gui-host')) return;

    const host = elem('div');
    host.id = 'amnesis-gui-host';
    host.style.cssText =
      'position:fixed;inset:0;pointer-events:none;z-index:2147483647';

    document.documentElement.append(host);
    root = host.attachShadow({ mode: 'open' });

    root.innerHTML = `
      <style>
        *,*::before,*::after{box-sizing:border-box}
        :host{font:12px Tahoma,"Segoe UI",Arial,sans-serif;color:#dedede}
        button,input,select{font:inherit}

        #window{position:fixed;width:1040px;height:680px;max-width:calc(100vw - 20px);max-height:calc(100vh - 20px);display:none;overflow:hidden;pointer-events:auto;isolation:isolate;--accent:#d52232;background:#111;border:1px solid var(--accent);border-radius:0;box-shadow:0 18px 55px #000c}
        #window::before{content:"";position:absolute;z-index:5;left:0;right:0;top:0;height:1px;background:#ff5260;pointer-events:none}
        #window.open{display:flex;flex-direction:column;animation:amnesis-open .16s ease-out}
        #window.compact{height:500px}
        #window.ui-blur{background:#101010ed;-webkit-backdrop-filter:blur(10px);backdrop-filter:blur(10px)}
        #window.no-ui-animation,#window.no-ui-animation *{animation:none!important;transition:none!important}
        @keyframes amnesis-open{from{opacity:0;transform:translateY(8px) scale(.988)}to{opacity:1;transform:none}}

        #header{height:48px;flex:none;display:grid;grid-template-columns:180px minmax(0,1fr) 112px;align-items:stretch;padding:0;background:#171717;border-bottom:1px solid var(--accent);cursor:grab;user-select:none}
        #brand{display:flex;align-items:center;gap:10px;padding:0 12px;border-right:1px solid var(--accent)}
        #brand .mark{position:relative;width:30px;height:30px;display:block;overflow:hidden;border:1px solid #7c2028;border-radius:0;background:#090909;box-shadow:none}
        #brand .mark img{display:block;width:100%;height:100%;object-fit:cover}
        #brand div{display:flex;flex-direction:column;gap:2px}
        #brand strong{font:700 14px/1 Tahoma,Arial,sans-serif;letter-spacing:2px;color:#fff}
        #brand small{font:400 8px/1 Consolas,monospace;letter-spacing:1px;color:#8b8b8b}
        #status{display:flex;align-items:center;justify-content:center;gap:7px;border-left:1px solid var(--accent);background:#131313;color:#aaa;font:400 9px/1 Consolas,monospace;letter-spacing:.5px}
        #status b{width:6px;height:6px;background:var(--accent);box-shadow:none}

        #body{display:flex;flex:1;min-height:0;background:#101010}
        #sidebar{min-width:0;height:47px;display:flex;align-items:stretch;overflow-x:auto;background:#151515}
        #sidebar::before,#sidebar::after{display:none}
        #content{flex:1;min-width:0;padding:10px 12px 16px;overflow:auto;background:#101010}
        .panel-grid{column-count:3;column-gap:9px}

        ::-webkit-scrollbar{width:6px;height:6px}
        ::-webkit-scrollbar-thumb{background:#67202a;border-radius:0}
        ::-webkit-scrollbar-thumb:hover{background:var(--accent)}
        ::-webkit-scrollbar-track{background:transparent}

        .nav{display:flex;align-items:center;justify-content:center;gap:7px;min-width:86px;height:47px;border:0;border-right:1px solid #3b171b;background:#151515;color:#c8c8c8;text-align:center;padding:0 10px;margin:0;font:400 11px/1 Tahoma,Arial,sans-serif;cursor:pointer;transition:background .1s,color .1s}
        .nav svg{width:14px;height:14px;flex:none;fill:none;stroke:currentColor;stroke-width:1.8;stroke-linecap:square;stroke-linejoin:miter}
        .nav:hover{color:#fff;background:#1d1d1d}
        .nav.active{color:#fff;background:#202020;box-shadow:inset 0 -2px 0 var(--accent)}
        .nav.active svg{color:var(--accent);filter:none}

        h2{font:400 13px/1 Tahoma,Arial,sans-serif;color:#fff;margin:0 0 8px;padding:6px 8px;background:#181818;border:1px solid #4d1b21;border-left:3px solid var(--accent);letter-spacing:.2px}
        h2::before,h2::after{display:none}

        .section{position:relative;display:inline-block;width:100%;break-inside:avoid;margin:0 0 9px;padding:8px 9px 7px;background:#151515;border:1px solid #6f2029;border-radius:0;box-shadow:none;overflow:hidden}
        .section::before{display:none}
        .section strong{font-size:11px;font-weight:400;letter-spacing:0;text-transform:none;color:var(--accent)}
        .rage-area-options[hidden]{display:none!important}
        .rage-area-picker{margin:8px 0}
        .rage-area-options{padding:6px 10px;border:1px solid #303641;background:#0c0f14;border-radius:5px;margin-top:6px}
        .section.collapsible{cursor:pointer;transition:border-color .1s,background .1s}
        .section.collapsible:hover{border-color:#b42a38;background:#171717;transform:none}
        .section.collapsible .section-title{display:flex;justify-content:space-between;align-items:center;gap:12px;outline:none}
        .section.collapsible .section-title::after{content:"+";font:400 13px/1 Consolas,monospace;color:#888}
        .section.collapsible:not(.collapsed) .section-title::after{content:"-";color:var(--accent)}
        .section.collapsible .section-title:focus-visible{outline:1px solid var(--accent);outline-offset:5px}
        .section.collapsible.collapsed>:not(.section-title){display:none!important}
        .bind-title-actions{display:flex;align-items:center;margin-left:auto;margin-right:7px}
        .amnesis-bind{font:400 8px/1 Consolas,monospace;letter-spacing:.3px;background:#111;border:1px solid #75222b;border-radius:0;color:#ccc;padding:4px 6px;min-width:42px;max-width:105px;cursor:pointer!important;white-space:nowrap;box-shadow:none}
        .amnesis-bind:hover{background:#211417;color:#fff;border-color:var(--accent)}
        .amnesis-bind.listening{background:color-mix(in srgb,var(--accent) 20%,#15181e);border-color:var(--accent);color:#fff;animation:amnesis-pulse .8s infinite alternate}
        @keyframes amnesis-pulse{from{opacity:.55}to{opacity:1}}
        .section-title .amnesis-bind{font-size:9px}

        .section.collapsible label,.section.collapsible button,.section.collapsible input,.section.collapsible select{cursor:pointer}
        .desc{color:#777;font-size:9px;line-height:1.35;margin:3px 0 6px;max-width:98%}
        .world-subheading{margin:16px 0 6px;padding-top:12px;border-top:1px solid #282d37;color:#dde1e9;font-size:11px;font-weight:700;letter-spacing:1px;text-transform:uppercase}
        .note{color:#626976;font-size:10px;line-height:1.5;margin:7px 0 0;padding-top:9px;border-top:1px solid #242933}
        p{color:#717887;line-height:1.5;font-size:11px}

        .field{display:flex;align-items:center;justify-content:space-between;gap:8px;min-height:27px;padding:4px 0;color:#ddd;border-top:1px solid #272727;font-size:10px}
        .cols{display:grid;grid-template-columns:minmax(0,1fr) minmax(0,1fr);column-gap:12px}

        input[type=checkbox]{-webkit-appearance:none;appearance:none;width:14px;height:14px;border-radius:0;background:#111;border:1px solid #8b2731;position:relative;cursor:pointer;flex:none;margin:0;outline:none}
        input[type=checkbox]::after{content:"";position:absolute;inset:3px;background:transparent}
        input[type=checkbox]:checked{background:#191919;border-color:#f03a4a}
        input[type=checkbox]:checked::after{background:var(--accent);box-shadow:none}

        input[type=range]{-webkit-appearance:none;appearance:none;height:7px;border:1px solid #8b2731;border-radius:0;background:#202020;outline:none;cursor:pointer;margin:0}
        input[type=range]::-webkit-slider-thumb{-webkit-appearance:none;width:8px;height:13px;border-radius:0;background:var(--accent);border:1px solid #ff6975;cursor:pointer;box-shadow:none}
        input[type=range]::-moz-range-thumb{width:10px;height:14px;border-radius:2px;background:#f4f6fa;border:2px solid var(--accent);cursor:pointer}

        select{appearance:none;-webkit-appearance:none;min-width:105px;max-width:190px;height:23px;padding:0 24px 0 7px;border:1px solid #7c232d;border-radius:0;background-color:#111;color:#eee;font:400 10px Tahoma,Arial,sans-serif;outline:none;cursor:pointer;color-scheme:dark;background-image:url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='12' height='8' viewBox='0 0 12 8'%3E%3Cpath d='m1 1 5 5 5-5' fill='none' stroke='%23d52232' stroke-width='1.6'/%3E%3C/svg%3E");background-repeat:no-repeat;background-position:right 7px center}
        select:hover{border-color:#535b69;background-color:#12161d}
        select:focus-visible{border-color:var(--accent);box-shadow:0 0 0 2px color-mix(in srgb,var(--accent) 20%,transparent)}
        select option{background:#11141a;color:#eceff4}
        .field>span:first-child{min-width:0;overflow-wrap:anywhere}
        .field>select,.field>input[type=range]{flex-shrink:0}
        input[type=number],input[type=search],input[type=text]{background:#111;color:#eee;border:1px solid #75222b;border-radius:0;outline:none;font:10px Consolas,monospace;padding:0 6px;height:23px}
        input[type=number]{width:82px;text-align:right}
        input[type=search]{width:100%;height:34px;margin:4px 0 0}
        input[type=number]:focus,input[type=search]:focus,input[type=text]:focus{border-color:var(--accent)}
        input[type=text]{width:210px;max-width:58%}
        input[type=color]{width:22px;height:18px;padding:1px;border:1px solid #8b2731;border-radius:0;background:#111;cursor:pointer;margin-left:4px}

        .modes{display:inline-flex;margin-top:4px;background:#111;border:1px solid #75222b;border-radius:0;padding:1px}
        .mode{background:transparent;border:0;color:#999;padding:5px 11px;border-radius:0;font:400 9px Tahoma,Arial,sans-serif;letter-spacing:0;cursor:pointer}
        .mode:hover{color:#c8ced8}
        .mode.selected{background:var(--accent);color:#fff;box-shadow:none}

        .dots{display:flex;gap:9px;align-items:center}
        .dot{width:15px;height:15px;border-radius:0;border:1px solid #7c232d;padding:0;cursor:pointer;transform:none}
        .dot.on{outline:1px solid #fff;outline-offset:1px}

        .action{background:#161616;color:#ddd;border:1px solid #75222b;border-radius:0;padding:5px 10px;margin:7px 5px 0 0;cursor:pointer;font:400 9px Tahoma,Arial,sans-serif;letter-spacing:.2px;text-transform:none}
        .action:hover{background:#261519;border-color:var(--accent);color:#fff}

        .empty{display:flex;flex-direction:column;align-items:center;justify-content:center;gap:7px;padding:32px 10px;color:#626976;font-size:11px;text-align:center}
        .empty svg{width:28px;height:28px;fill:none;stroke:currentColor;stroke-width:1.6;stroke-linecap:round;stroke-linejoin:round;color:var(--accent)}
        .empty b{font-weight:650;color:#aeb4c0}

        @media(max-width:1000px){.panel-grid{column-count:2}}
        @media(max-width:760px){
          #header{height:52px;padding:0}
          #brand small{display:none}
          #status span{font-size:0}
          #status span::after{content:"RSHIFT";font-size:9px}
          #header{grid-template-columns:145px minmax(0,1fr) 74px}
          #brand{padding:0 8px}.nav{min-width:42px}
          .nav{width:auto;height:46px;margin:0;padding:0 10px}
          .nav span{display:none}
          #content{padding:16px 12px 22px}
          .panel-grid{column-count:1}
          .cols{grid-template-columns:1fr}
        }

      </style>
      <div id="window">
        <div id="header">
          <div id="brand">
            <span class="mark"><img src="https://i.ibb.co/2YnYg7Ps/aasaf.png" alt="AMNESIS"></span>
            <div><strong>AMNESIS</strong><small>CONTROL MATRIX</small></div>
          </div>
          <aside id="sidebar"></aside>
          <div id="status"><b></b><span>RIGHT SHIFT / MENU</span></div>
        </div>
        <div id="body">
          <main id="content"></main>
        </div>
      </div>
    `;

    box = root.getElementById('window');
    nav = root.getElementById('sidebar');
    page = root.getElementById('content');

    box.style.setProperty('--accent', cfg.accent);
    box.classList.toggle('compact', cfg.compact);

    const bar = root.getElementById('header');

    bar.addEventListener('pointerdown', e => {
      if (e.button !== 0 || e.target.closest('button,input,select')) return;

      drag = {
        x: e.clientX,
        y: e.clientY,
        left: cfg.x,
        top: cfg.y
      };

      bar.setPointerCapture(e.pointerId);
      e.preventDefault();
    });

    bar.addEventListener('pointermove', e => {
      if (!drag) return;
      cfg.x = drag.left + e.clientX - drag.x;
      cfg.y = drag.top + e.clientY - drag.y;
      position();
    });

    const end = () => {
      drag = null;
      save();
    };

    bar.addEventListener('pointerup', end);
    bar.addEventListener('pointercancel', end);

    root.addEventListener('wheel', e => e.stopPropagation(), {
      passive: true
    });

    renderNav();
    renderPage();
    position();
    initESP();

  }

  window.addEventListener('resize', () => {
    if (box) {
      position();
      save();
    }
  });

  if (document.readyState === 'loading') {
    document.addEventListener('DOMContentLoaded', initialize, {
      once: true
    });
  } else {
    initialize();
  }
})();
