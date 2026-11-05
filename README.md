
extends WorldEnvironment
# Смена времени суток. Повесь этот скрипт на свой узел WorldEnvironment.
#
# Игровое время идёт по кругу 0-24 часа. Между утром, днём, вечером и ночью окружение
# (свет от неба, цвет неба или фона, туман, экспозиция) переходит плавно.
# Сейчас у тебя настроена ночь, поэтому ночной вид берётся из ТЕКУЩИХ настроек WorldEnvironment
# и не меняется. Утро, день и вечер имеют готовые значения по умолчанию, которые можно заменить
# своими ресурсами TimeOfDayPreset в инспекторе.
#
# Если назначить sun_light (DirectionalLight3D), солнце днём и луна ночью будут двигаться по небу.
# Если не назначать, свет от DirectionalLight скрипт не трогает.

signal period_changed(period: StringName)
signal day_passed(day_number: int)

@export_group("Time")
## Сколько реальных минут длятся ПОЛНЫЕ игровые сутки (24 игровых часа).
@export_range(0.1, 240.0, 0.1) var day_length_minutes: float = 24.0
## С какого часа начинается игра (0 = полночь, 12 = полдень).
@export_range(0.0, 24.0, 0.01) var start_hour: float = 0.0
## Остановить время.
@export var running: bool = true
## Текущий час. Можно менять прямо во время игры (вкладка Remote), чтобы проверить вид любого времени.
@export_range(0.0, 24.0, 0.01) var current_hour: float = 0.0

@export_group("Date (для часов в игре)")
## Игровая дата на старте. Каждые прошедшие игровые сутки добавляют день. Часы на компьютере и
## надписи на камерах берут время и дату отсюда, а не с твоего реального компьютера.
@export var start_year: int = 2004
@export_range(1, 12, 1) var start_month: int = 10
@export_range(1, 31, 1) var start_day: int = 12

@export_group("Key hours")
## Ночь заканчивается, начинается рассвет.
@export_range(0.0, 24.0, 0.1) var night_end_hour: float = 5.0
@export_range(0.0, 24.0, 0.1) var morning_hour: float = 7.0
@export_range(0.0, 24.0, 0.1) var noon_hour: float = 13.0
@export_range(0.0, 24.0, 0.1) var evening_hour: float = 19.0
## Вечер заканчивается, наступает полная ночь.
@export_range(0.0, 24.0, 0.1) var night_start_hour: float = 21.0

@export_group("Presets (необязательно)")
## Если пусто, берутся готовые значения по умолчанию.
@export var morning_preset: TimeOfDayPreset
@export var noon_preset: TimeOfDayPreset
@export var evening_preset: TimeOfDayPreset
## Если пусто, ночь берётся из текущих настроек WorldEnvironment (как сейчас).
@export var night_preset: TimeOfDayPreset

@export_group("Sky")
## Менять цвет неба (фона). Если у WorldEnvironment режим фона Clear Color, он при старте игры
## сам переключается на Color с тем же цветом, чтобы небо могло меняться (ночью ничего не меняется).
@export var control_sky_color: bool = true

@export_group("Skybox (небо)")
## Динамическое небо: градиент по цветам из пресетов, звёзды ночью и (по желанию) панорамы.
## Если выключить, остаётся прежний плоский цвет фона.
@export var use_dynamic_sky: bool = true
## Панорамы неба для каждого времени суток (необязательно). Формат 2:1 (equirectangular), без солнца и луны.
## Можно задать не все: для времён без картинки небо остаётся процедурным. Между картинками небо плавно перетекает.
@export var sky_morning_texture: Texture2D
@export var sky_day_texture: Texture2D
@export var sky_evening_texture: Texture2D
@export var sky_night_texture: Texture2D
## Насколько панорамы перекрывают процедурное небо (1 = полностью).
@export_range(0.0, 1.0, 0.01) var panorama_strength: float = 1.0
## Яркость панорам.
@export_range(0.0, 3.0, 0.01) var panorama_brightness: float = 1.0
## Поворот всего неба вокруг вертикали (градусы): можно подвернуть панораму и звёзды под свои окна.
@export_range(-180.0, 180.0, 1.0) var sky_rotation_degrees: float = 0.0

@export_group("Procedural Clouds (процедурные облака)")
## Включить процедурные облака поверх градиентного неба.
@export var clouds_enabled: bool = true
## Общая плотность облаков: выше значение — больше облачных областей.
@export_range(0.0, 1.0, 0.01) var clouds_coverage: float = 0.52
## Масштаб облачных форм: меньше — крупнее облака.
@export_range(1.0, 20.0, 0.1) var clouds_scale: float = 5.0
## Скорость движения облаков. 0 — неподвижны.
@export_range(0.0, 0.1, 0.001) var clouds_speed: float = 0.008
## Прозрачность/сила облачного слоя.
@export_range(0.0, 1.0, 0.01) var clouds_opacity: float = 0.65
## Мягкость краёв облаков.
@export_range(0.01, 0.4, 0.01) var clouds_softness: float = 0.12
## Нижняя граница облаков над горизонтом (по вертикальному направлению неба).
@export_range(-0.1, 0.5, 0.01) var clouds_min_height: float = 0.02
## Верхняя граница облачного слоя.
@export_range(0.2, 1.0, 0.01) var clouds_max_height: float = 0.82
## Автоматически подбирать цвет облаков из текущего пресета времени суток.
@export var clouds_auto_tint: bool = true
## Ручной цвет облаков, если автоматический подбор отключён.
@export var clouds_tint: Color = Color(0.92, 0.94, 1.0)
## Показывать процедурные облака поверх назначенных панорам неба.
@export var clouds_over_panorama: bool = false
## Использовать объёмный ray-marching вместо плоского процедурного слоя.
@export var clouds_volumetric_enabled: bool = true
## Число шагов ray-marching. Меньше — быстрее, больше — мягче и детальнее.
@export_range(8, 64, 4) var clouds_raymarch_steps: int = 24
## Общая плотность объёма. Подбирается в зависимости от покрытия.
@export_range(0.1, 4.0, 0.05) var clouds_volume_density: float = 1.15
## Высота нижней границы облаков в условных мировых метрах.
@export_range(50.0, 800.0, 10.0) var clouds_base_height: float = 180.0
## Высота верхней границы облаков в условных мировых метрах.
@export_range(100.0, 1600.0, 10.0) var clouds_top_height: float = 520.0
## Сила светового рассеивания в облаках.
@export_range(0.0, 2.0, 0.05) var clouds_light_strength: float = 0.75

@export_group("Stars (звёзды)")
@export var stars_enabled: bool = true
## Доля "ячеек" неба, в которых есть звезда (больше значение - больше звёзд).
@export_range(0.0, 0.3, 0.005) var stars_density: float = 0.04
@export_range(0.0, 6.0, 0.05) var stars_brightness: float = 2.5
## Показывать звёзды и поверх панорамы (по умолчанию скрыты там, где панорама, чтобы звёзды не двоились).
@export var stars_over_panorama: bool = false

@export_group("Sky sprites (солнце и луна)")
## Картинка солнца и луны. PNG с прозрачным фоном (альфа-каналом). Можно оставить пустым.
@export var sun_texture: Texture2D
@export var moon_texture: Texture2D
## Видимый размер на небе в градусах. Настоящие солнце и луна около 0.5, для красоты обычно 5-10.
@export_range(0.5, 30.0, 0.1) var sun_size_degrees: float = 11.0
@export_range(0.5, 30.0, 0.1) var moon_size_degrees: float = 6.0
## На каком расстоянии от камеры висят солнце и луна (метры). Должно быть меньше дальности камеры (Far).
@export_range(50.0, 3000.0, 10.0) var sky_distance: float = 400.0
## Чёткие пиксели без сглаживания (для маленьких картинок в стиле PS1).
@export var pixel_style: bool = false
## Цвет солнца днём и на закате (на закате оно краснеет).
@export var sun_tint: Color = Color(1.0, 0.95, 0.8)
@export var sunset_tint: Color = Color(1.0, 0.5, 0.25)
@export var moon_tint: Color = Color(0.85, 0.9, 1.0)

@export_group("Lighting (тени и интерьер)")
## Включить тени у sun_light (днём это солнце, ночью луна). У источников света тени по умолчанию выключены.
@export var sun_shadows: bool = true
## До какого расстояния от камеры считаются тени солнца (метры). Меньше - чётче тени и быстрее.
@export_range(10.0, 500.0, 1.0) var sun_shadow_distance: float = 80.0
## Во сколько раз ослабить ОБЩИЙ фоновый свет днём. Он одинаков везде, даже в комнатах без окон и под крышей,
## поэтому чем он слабее, тем заметнее, что в комнате без окна темно, а у окна светло.
## Действует, только когда назначен sun_light. 1 - не менять.
@export_range(0.0, 1.0, 0.01) var daytime_ambient_scale: float = 0.45

@export_group("Sun / Moon")
## DirectionalLight3D, который будет изображать солнце днём и луну ночью. Можно оставить пустым.
@export var sun_light: DirectionalLight3D
## Куда сдвинуто движение солнца по горизонту (градусы).
@export var sun_yaw_degrees: float = -30.0
## Максимальная высота солнца над горизонтом в полдень.
@export_range(10.0, 90.0, 1.0) var max_sun_elevation_degrees: float = 65.0

var day_number: int = 1

var _morning: TimeOfDayPreset
var _noon: TimeOfDayPreset
var _evening: TimeOfDayPreset
var _night: TimeOfDayPreset
var _work: TimeOfDayPreset = TimeOfDayPreset.new()
var _period: StringName = &""
var _ready_ok: bool = false
var _sun_sprite: Sprite3D
var _moon_sprite: Sprite3D
var _sky_material: ShaderMaterial
## Вес каждой части суток для смешивания панорам: x - утро, y - день, z - вечер, w - ночь.
var _sky_weights: Vector4 = Vector4(0.0, 0.0, 0.0, 1.0)

const SKY_SHADER_CODE := """
shader_type sky;
// Динамическое небо: градиент (цвета из пресетов), звёзды и необязательные панорамы по времени суток.

uniform vec3 sky_top : source_color = vec3(0.3, 0.5, 0.9);
uniform vec3 sky_horizon : source_color = vec3(0.7, 0.8, 0.95);
uniform vec3 ground_color : source_color = vec3(0.4, 0.4, 0.4);
uniform float gradient_energy = 1.0;

uniform float star_visibility = 0.0;
uniform float star_density = 0.04;
uniform float star_brightness = 2.5;
uniform bool stars_over_panorama = false;

uniform sampler2D tex_morning : source_color, filter_linear, hint_default_black;
uniform sampler2D tex_day : source_color, filter_linear, hint_default_black;
uniform sampler2D tex_evening : source_color, filter_linear, hint_default_black;
uniform sampler2D tex_night : source_color, filter_linear, hint_default_black;
uniform vec4 tex_present = vec4(0.0);
uniform vec4 tex_weights = vec4(0.0, 0.0, 0.0, 1.0);
uniform float panorama_strength = 1.0;
uniform float panorama_brightness = 1.0;

// Процедурный облачный слой. Он рисуется внутри sky shader и не меняет WorldEnvironment.
uniform bool clouds_enabled = true;
uniform float clouds_coverage = 0.52;
uniform float clouds_scale = 5.0;
uniform float clouds_speed = 0.008;
uniform float clouds_opacity = 0.65;
uniform float clouds_softness = 0.12;
uniform float clouds_min_height = 0.02;
uniform float clouds_max_height = 0.82;
uniform vec3 clouds_tint : source_color = vec3(0.92, 0.94, 1.0);
uniform bool clouds_over_panorama = false;
uniform bool clouds_volumetric_enabled = true;
uniform int clouds_raymarch_steps = 24;
uniform float clouds_volume_density = 1.15;
uniform float clouds_base_height = 180.0;
uniform float clouds_top_height = 520.0;
uniform float clouds_light_strength = 0.75;
uniform vec3 clouds_light_direction = vec3(0.0, 1.0, 0.0);
uniform vec3 clouds_light_color : source_color = vec3(1.0, 0.95, 0.85);

float hash13(vec3 p) {
	p = fract(p * 0.1031);
	p += dot(p, p.zyx + 31.32);
	return fract((p.x + p.y) * p.z);
}

float value_noise(vec3 p) {
	vec3 cell = floor(p);
	vec3 f = fract(p);
	f = f * f * (3.0 - 2.0 * f);

	float n000 = hash13(cell);
	float n100 = hash13(cell + vec3(1.0, 0.0, 0.0));
	float n010 = hash13(cell + vec3(0.0, 1.0, 0.0));
	float n110 = hash13(cell + vec3(1.0, 1.0, 0.0));
	float n001 = hash13(cell + vec3(0.0, 0.0, 1.0));
	float n101 = hash13(cell + vec3(1.0, 0.0, 1.0));
	float n011 = hash13(cell + vec3(0.0, 1.0, 1.0));
	float n111 = hash13(cell + vec3(1.0, 1.0, 1.0));

	float x00 = mix(n000, n100, f.x);
	float x10 = mix(n010, n110, f.x);
	float x01 = mix(n001, n101, f.x);
	float x11 = mix(n011, n111, f.x);
	return mix(mix(x00, x10, f.y), mix(x01, x11, f.y), f.z);
}

float cloud_noise(vec3 p) {
	float result = 0.0;
	float amplitude = 0.5;
	float total = 0.0;
	for (int octave = 0; octave < 4; octave++) {
		result += value_noise(p) * amplitude;
		total += amplitude;
		p *= 2.03;
		amplitude *= 0.5;
	}
	return result / max(total, 0.001);
}

float cloud_layer(vec3 dir) {
	if (!clouds_enabled) {
		return 0.0;
	}
	// Legacy inexpensive cloud layer retained as fallback.
	vec3 p = dir * clouds_scale;
	p.x += TIME * clouds_speed;
	p.z += TIME * clouds_speed * 0.37;
	float noise_value = cloud_noise(p);
	float threshold = 1.0 - clouds_coverage;
	float shape = smoothstep(threshold, threshold + clouds_softness, noise_value);
	float lower_fade = smoothstep(clouds_min_height, clouds_min_height + 0.08, dir.y);
	float upper_fade = 1.0 - smoothstep(clouds_max_height - 0.12, clouds_max_height, dir.y);
	return shape * lower_fade * upper_fade;
}

// Approximate volumetric cloud slab. Density is integrated along the view ray.
// Fixed upper loop bound is shader-friendly; Inspector controls the active step count.
vec4 volumetric_clouds(vec3 dir) {
	if (!clouds_enabled || !clouds_volumetric_enabled || dir.y <= 0.015) {
		return vec4(0.0);
	}
	float base_h = max(clouds_base_height, 1.0);
	float top_h = max(clouds_top_height, base_h + 10.0);
	float t0 = base_h / dir.y;
	float t1 = top_h / dir.y;
	// Prevent grazing rays from producing unbounded distances.
	t0 = min(t0, 12000.0);
	t1 = min(t1, 12000.0);
	if (t1 <= t0) {
		return vec4(0.0);
	}
	int steps = clamp(clouds_raymarch_steps, 8, 64);
	float step_len = (t1 - t0) / float(steps);
	float transmittance = 1.0;
	float accumulated_light = 0.0;
	float height_span = max(top_h - base_h, 1.0);
	float coverage_threshold = 0.72 - clouds_coverage * 0.42;
	vec3 light_dir = normalize(clouds_light_direction);
	for (int i = 0; i < 64; i++) {
		if (i >= steps || transmittance < 0.025) {
			break;
		}
		float distance_along = t0 + (float(i) + 0.5) * step_len;
		vec3 world_sample = dir * distance_along;
		float normalized_height = clamp((world_sample.y - base_h) / height_span, 0.0, 1.0);
		// 3D animated noise: horizontal coordinates provide cloud depth and vertical coordinate shapes billows.
		vec3 noise_pos = vec3(world_sample.xz * 0.0018 * clouds_scale,
			(normalized_height * 2.8 + TIME * clouds_speed * 0.12) * clouds_scale * 0.22);
		noise_pos.x += TIME * clouds_speed * 0.8;
		noise_pos.z += TIME * clouds_speed * 0.35;
		float n = cloud_noise(noise_pos);
		float profile = smoothstep(0.0, 0.18, normalized_height) * (1.0 - smoothstep(0.72, 1.0, normalized_height));
		float density_shape = smoothstep(coverage_threshold, coverage_threshold + max(clouds_softness, 0.06), n);
		float density = density_shape * profile * clouds_volume_density;
		float sample_alpha = 1.0 - exp(-density * step_len * 0.0022);
		// Simple forward-scattering approximation: cloud edges brighten toward the light.
		float phase = 0.35 + 0.65 * pow(max(dot(normalize(dir), light_dir), 0.0), 2.0);
		accumulated_light += transmittance * sample_alpha * phase;
		transmittance *= (1.0 - sample_alpha);
	}
	float opacity = 1.0 - transmittance;
	float lighting = accumulated_light / max(opacity, 0.001);
	vec3 lit_color = mix(clouds_tint * 0.58, clouds_light_color, clamp(lighting * clouds_light_strength, 0.0, 1.0));
	return vec4(lit_color, opacity * clouds_opacity);
}

vec3 procedural_sky(vec3 dir) {
	float h = dir.y;
	vec3 col;
	if (h >= 0.0) {
		float t = pow(1.0 - clamp(h, 0.0, 1.0), 3.0);
		col = mix(sky_top, sky_horizon, t);
	} else {
		float t = pow(clamp(-h * 4.0, 0.0, 1.0), 0.6);
		col = mix(sky_horizon, ground_color, t);
	}
	return col * gradient_energy;
}

vec3 star_field(vec3 dir) {
	if (star_visibility <= 0.001 || dir.y < -0.02) {
		return vec3(0.0);
	}
	vec3 p = dir * 120.0;
	vec3 cell = floor(p);
	vec3 local = fract(p) - 0.5;
	float h = hash13(cell);
	if (h > star_density) {
		return vec3(0.0);
	}
	vec3 offset = (vec3(hash13(cell + 17.0), hash13(cell + 31.0), hash13(cell + 47.0)) - 0.5) * 0.6;
	float d = length(local - offset);
	float size = 0.16 + 0.12 * hash13(cell + 5.0);
	float star = smoothstep(size, 0.0, d);
	float twinkle = 0.75 + 0.25 * sin(TIME * (1.5 + 3.0 * hash13(cell + 9.0)) + h * 50.0);
	vec3 tint = mix(vec3(0.8, 0.85, 1.0), vec3(1.0, 0.9, 0.75), hash13(cell + 3.0));
	return tint * star * twinkle * star_brightness * star_visibility;
}

void sky() {
	vec3 dir = EYEDIR;
	vec3 col = procedural_sky(dir);

	// Панорамы: смешиваем те, что заданы, по весам времени суток
	vec2 puv = vec2(fract(atan(dir.x, -dir.z) / TAU + 0.5), acos(clamp(dir.y, -1.0, 1.0)) / PI);
	vec3 pan = vec3(0.0);
	float coverage = 0.0;
	if (tex_present.x > 0.5) {
		pan += textureLod(tex_morning, puv, 0.0).rgb * tex_weights.x;
		coverage += tex_weights.x;
	}
	if (tex_present.y > 0.5) {
		pan += textureLod(tex_day, puv, 0.0).rgb * tex_weights.y;
		coverage += tex_weights.y;
	}
	if (tex_present.z > 0.5) {
		pan += textureLod(tex_evening, puv, 0.0).rgb * tex_weights.z;
		coverage += tex_weights.z;
	}
	if (tex_present.w > 0.5) {
		pan += textureLod(tex_night, puv, 0.0).rgb * tex_weights.w;
		coverage += tex_weights.w;
	}
	float pan_amount = 0.0;
	if (coverage > 0.001) {
		pan = pan / coverage * panorama_brightness;
		pan_amount = clamp(coverage, 0.0, 1.0) * panorama_strength;
		col = mix(col, pan, pan_amount);
	}

	float panorama_cloud_mask = clouds_over_panorama ? 1.0 : (1.0 - pan_amount);
	if (clouds_volumetric_enabled) {
		vec4 cloud_volume = volumetric_clouds(dir);
		float cloud_alpha = clamp(cloud_volume.a * panorama_cloud_mask, 0.0, 1.0);
		col = mix(col, cloud_volume.rgb, cloud_alpha);
	} else {
		float cloud_amount = cloud_layer(dir) * clouds_opacity * panorama_cloud_mask;
		col = mix(col, clouds_tint, clamp(cloud_amount, 0.0, 1.0));
	}

	float star_mask = stars_over_panorama ? 1.0 : (1.0 - pan_amount);
	col += star_field(dir) * star_mask;
	COLOR = col;
}
"""


func _ready() -> void:
	# Другие системы (часы на мониторе, надписи на камерах) находят игровое время по этой группе.
	add_to_group("game_clock")
	if environment == null:
		push_error("day_night_cycle.gd: у WorldEnvironment не задан Environment.")
		return
	_night = night_preset if night_preset != null else _capture_current_as_preset()
	_prepare_background()
	_setup_dynamic_sky()
	_morning = morning_preset if morning_preset != null else _make_default(&"morning")
	_noon = noon_preset if noon_preset != null else _make_default(&"noon")
	_evening = evening_preset if evening_preset != null else _make_default(&"evening")
	current_hour = start_hour
	if sun_light != null and sun_shadows:
		sun_light.shadow_enabled = true
		sun_light.directional_shadow_max_distance = sun_shadow_distance
	if sun_texture != null:
		_sun_sprite = _create_sky_sprite(sun_texture, sun_size_degrees)
	if moon_texture != null:
		_moon_sprite = _create_sky_sprite(moon_texture, moon_size_degrees)
	_ready_ok = true
	_apply()


func _process(delta: float) -> void:
	if not _ready_ok:
		return
	if running:
		var hours_per_second: float = 24.0 / (maxf(day_length_minutes, 0.01) * 60.0)
		current_hour += delta * hours_per_second
		if current_hour >= 24.0:
			current_hour = fmod(current_hour, 24.0)
			day_number += 1
			day_passed.emit(day_number)
	_apply()


## Установить время напрямую (час от 0 до 24).
func set_time(hour: float) -> void:
	current_hour = fposmod(hour, 24.0)
	if _ready_ok:
		_apply()


## Часы, минуты и секунды игрового времени.
func get_time_parts() -> Dictionary:
	var hours: int = int(current_hour)
	var minutes_float: float = (current_hour - float(hours)) * 60.0
	var minutes: int = int(minutes_float)
	var seconds: int = int((minutes_float - float(minutes)) * 60.0)
	return {"hour": hours, "minute": minutes, "second": seconds}


## Игровая дата: словарь с ключами year, month, day, weekday (0 = воскресенье).
func get_date_dict() -> Dictionary:
	var base: int = Time.get_unix_time_from_datetime_dict({
		"year": start_year, "month": start_month, "day": start_day,
		"hour": 0, "minute": 0, "second": 0
	})
	return Time.get_datetime_dict_from_unix_time(base + (day_number - 1) * 86400)


## Время в виде "ЧЧ:ММ".
func get_time_string() -> String:
	var hours: int = int(current_hour)
	var minutes: int = int((current_hour - float(hours)) * 60.0)
	return "%02d:%02d" % [hours, minutes]


## Текущая часть суток: &"night", &"morning", &"day" или &"evening".
func get_period() -> StringName:
	var h: float = current_hour
	if h >= night_start_hour or h < night_end_hour:
		return &"night"
	if h < (morning_hour + noon_hour) * 0.5:
		return &"morning"
	if h < (noon_hour + evening_hour) * 0.5:
		return &"day"
	return &"evening"


# =====================================================================================

func _apply() -> void:
	var keys: Array = _build_keys()
	var h: float = current_hour
	var from_key: Array = keys[0]
	var to_key: Array = keys[1]
	for i in range(keys.size() - 1):
		var a: Array = keys[i]
		var b: Array = keys[i + 1]
		if h >= float(a[0]) and h < float(b[0]):
			from_key = a
			to_key = b
			break
	var span: float = maxf(float(to_key[0]) - float(from_key[0]), 0.0001)
	var t: float = smoothstep(0.0, 1.0, (h - float(from_key[0])) / span)
	_blend(from_key[1], to_key[1], t)
	_sky_weights = _weights_for(int(from_key[2]), int(to_key[2]), t)
	_apply_environment(_work)
	_apply_sun(_work)
	_update_sky_bodies()
	_update_sky_shader(_work)

	var period: StringName = get_period()
	if period != _period:
		_period = period
		period_changed.emit(period)


# Ключевые точки суток: ночь, утро, день, вечер, ночь. Края замыкаются, чтобы ночь была ровной.
func _build_keys() -> Array:
	# Третье число: 0 - ночь, 1 - утро, 2 - день, 3 - вечер (нужно для смешивания панорам неба).
	return [
		[night_start_hour - 24.0, _night, 0],
		[night_end_hour, _night, 0],
		[morning_hour, _morning, 1],
		[noon_hour, _noon, 2],
		[evening_hour, _evening, 3],
		[night_start_hour, _night, 0],
		[night_end_hour + 24.0, _night, 0],
	]


func _blend(a: TimeOfDayPreset, b: TimeOfDayPreset, t: float) -> void:
	_work.ambient_color = a.ambient_color.lerp(b.ambient_color, t)
	_work.ambient_energy = lerpf(a.ambient_energy, b.ambient_energy, t)
	_work.background_color = a.background_color.lerp(b.background_color, t)
	_work.background_energy = lerpf(a.background_energy, b.background_energy, t)
	_work.sky_top_color = a.sky_top_color.lerp(b.sky_top_color, t)
	_work.sky_horizon_color = a.sky_horizon_color.lerp(b.sky_horizon_color, t)
	_work.ground_color = a.ground_color.lerp(b.ground_color, t)
	_work.fog_color = a.fog_color.lerp(b.fog_color, t)
	_work.fog_density = lerpf(a.fog_density, b.fog_density, t)
	_work.exposure = lerpf(a.exposure, b.exposure, t)
	_work.sun_color = a.sun_color.lerp(b.sun_color, t)
	_work.sun_energy = lerpf(a.sun_energy, b.sun_energy, t)


func _apply_environment(p: TimeOfDayPreset) -> void:
	var env: Environment = environment
	env.ambient_light_color = p.ambient_color
	var ambient_energy: float = p.ambient_energy
	if sun_light != null:
		# Днём общий фоновый свет слабее: основной свет в комнаты даёт солнце через окна, а под крышей темнее.
		var day_amount: float = 1.0 - _sky_weights.w
		ambient_energy *= lerpf(1.0, daytime_ambient_scale, day_amount)
	env.ambient_light_energy = ambient_energy
	env.fog_light_color = p.fog_color
	env.fog_density = p.fog_density
	env.tonemap_exposure = p.exposure
	if env.background_mode == Environment.BG_COLOR:
		env.background_color = p.background_color
		env.background_energy_multiplier = p.background_energy
	elif env.background_mode == Environment.BG_SKY:
		# У динамического неба яркость задаёт сам шейдер, поэтому общий множитель равен 1.
		env.background_energy_multiplier = 1.0 if _sky_material != null else p.background_energy
	var sky_material := _get_procedural_sky()
	if sky_material != null:
		sky_material.sky_top_color = p.sky_top_color
		sky_material.sky_horizon_color = p.sky_horizon_color
		sky_material.ground_horizon_color = p.sky_horizon_color
		sky_material.ground_bottom_color = p.ground_color


func _apply_sun(p: TimeOfDayPreset) -> void:
	if sun_light == null:
		return
	var elevation: float = _sun_elevation()
	var yaw: float = _sun_yaw()
	# Днём свет идёт от солнца, ночью от луны (она всегда напротив солнца).
	# У самого горизонта яркость падает до нуля, поэтому смена направления света незаметна.
	var day_weight: float = smoothstep(0.0, 10.0, elevation)
	var night_weight: float = smoothstep(0.0, 10.0, -elevation)
	if elevation >= 0.0:
		sun_light.rotation_degrees = Vector3(-elevation, yaw, 0.0)
		sun_light.light_color = p.sun_color
	else:
		sun_light.rotation_degrees = Vector3(elevation, yaw + 180.0, 0.0)
		sun_light.light_color = _night.sun_color
	sun_light.light_energy = p.sun_energy * day_weight + _night.sun_energy * night_weight


# Высота солнца над горизонтом в градусах (минус - под горизонтом): 0 в 6:00, максимум в 12:00, 0 в 18:00.
func _sun_elevation() -> float:
	return sin((current_hour - 6.0) / 24.0 * TAU) * max_sun_elevation_degrees


# Направление солнца по горизонту: оно восходит с одной стороны (sun_yaw - 90) и садится с противоположной
# (sun_yaw + 90), проходя в полдень через sun_yaw.
func _sun_yaw() -> float:
	return sun_yaw_degrees + (current_hour - 12.0) * 15.0


# --- Динамическое небо ---------------------------------------------------------

func _setup_dynamic_sky() -> void:
	if not use_dynamic_sky:
		return
	var shader := Shader.new()
	shader.code = SKY_SHADER_CODE
	_sky_material = ShaderMaterial.new()
	_sky_material.shader = shader
	_sky_material.set_shader_parameter("tex_morning", sky_morning_texture)
	_sky_material.set_shader_parameter("tex_day", sky_day_texture)
	_sky_material.set_shader_parameter("tex_evening", sky_evening_texture)
	_sky_material.set_shader_parameter("tex_night", sky_night_texture)
	_sky_material.set_shader_parameter("tex_present", Vector4(
		1.0 if sky_morning_texture != null else 0.0,
		1.0 if sky_day_texture != null else 0.0,
		1.0 if sky_evening_texture != null else 0.0,
		1.0 if sky_night_texture != null else 0.0
	))
	_sky_material.set_shader_parameter("clouds_enabled", clouds_enabled)
	_sky_material.set_shader_parameter("clouds_coverage", clouds_coverage)
	_sky_material.set_shader_parameter("clouds_scale", clouds_scale)
	_sky_material.set_shader_parameter("clouds_speed", clouds_speed)
	_sky_material.set_shader_parameter("clouds_opacity", clouds_opacity)
	_sky_material.set_shader_parameter("clouds_softness", clouds_softness)
	_sky_material.set_shader_parameter("clouds_min_height", clouds_min_height)
	_sky_material.set_shader_parameter("clouds_max_height", clouds_max_height)
	_sky_material.set_shader_parameter("clouds_over_panorama", clouds_over_panorama)
	_sky_material.set_shader_parameter("clouds_tint", clouds_tint)
	_sky_material.set_shader_parameter("clouds_volumetric_enabled", clouds_volumetric_enabled)
	_sky_material.set_shader_parameter("clouds_raymarch_steps", clouds_raymarch_steps)
	_sky_material.set_shader_parameter("clouds_volume_density", clouds_volume_density)
	_sky_material.set_shader_parameter("clouds_base_height", clouds_base_height)
	_sky_material.set_shader_parameter("clouds_top_height", clouds_top_height)
	_sky_material.set_shader_parameter("clouds_light_strength", clouds_light_strength)

	var sky := Sky.new()
	sky.sky_material = _sky_material
	# Небо влияет только на фон и отражения, поэтому радиационная карта может быть маленькой.
	sky.radiance_size = Sky.RADIANCE_SIZE_32
	environment.sky = sky
	environment.background_mode = Environment.BG_SKY


func _update_sky_shader(p: TimeOfDayPreset) -> void:
	if _sky_material == null:
		return
	_sky_material.set_shader_parameter("sky_top", p.sky_top_color)
	_sky_material.set_shader_parameter("sky_horizon", p.sky_horizon_color)
	_sky_material.set_shader_parameter("ground_color", p.ground_color)
	_sky_material.set_shader_parameter("gradient_energy", p.background_energy)

	# Облачные параметры берутся из Inspector, а оттенок плавно следует текущему пресету.
	_sky_material.set_shader_parameter("clouds_enabled", clouds_enabled)
	_sky_material.set_shader_parameter("clouds_coverage", clouds_coverage)
	_sky_material.set_shader_parameter("clouds_scale", clouds_scale)
	_sky_material.set_shader_parameter("clouds_speed", clouds_speed)
	_sky_material.set_shader_parameter("clouds_opacity", clouds_opacity)
	_sky_material.set_shader_parameter("clouds_softness", clouds_softness)
	_sky_material.set_shader_parameter("clouds_min_height", clouds_min_height)
	_sky_material.set_shader_parameter("clouds_max_height", clouds_max_height)
	_sky_material.set_shader_parameter("clouds_over_panorama", clouds_over_panorama)
	_sky_material.set_shader_parameter("clouds_volumetric_enabled", clouds_volumetric_enabled)
	_sky_material.set_shader_parameter("clouds_raymarch_steps", clouds_raymarch_steps)
	_sky_material.set_shader_parameter("clouds_volume_density", clouds_volume_density)
	_sky_material.set_shader_parameter("clouds_base_height", clouds_base_height)
	_sky_material.set_shader_parameter("clouds_top_height", clouds_top_height)
	_sky_material.set_shader_parameter("clouds_light_strength", clouds_light_strength)
	var sun_elevation_now: float = _sun_elevation()
	var sun_yaw_now: float = _sun_yaw()
	var cloud_light_dir: Vector3 = Basis.from_euler(Vector3(deg_to_rad(-sun_elevation_now), deg_to_rad(sun_yaw_now), 0.0)).z.normalized()
	if sun_elevation_now < 0.0:
		cloud_light_dir = Basis.from_euler(Vector3(deg_to_rad(sun_elevation_now), deg_to_rad(sun_yaw_now + 180.0), 0.0)).z.normalized()
	_sky_material.set_shader_parameter("clouds_light_direction", cloud_light_dir)
	var cloud_light_color: Color = p.sun_color.lerp(_night.sun_color, _sky_weights.w)
	_sky_material.set_shader_parameter("clouds_light_color", cloud_light_color)
	var cloud_color: Color = clouds_tint
	if clouds_auto_tint:
		cloud_color = p.sky_horizon_color.lerp(p.sky_top_color, 0.35)
		# Ночью облака становятся холоднее и темнее, не вмешиваясь в параметры света.
		cloud_color = cloud_color.lerp(Color(0.24, 0.29, 0.42), _sky_weights.w * 0.75)
	_sky_material.set_shader_parameter("clouds_tint", cloud_color)

	# Звёзды появляются в сумерках и полностью видны ночью.
	var star_amount: float = smoothstep(0.35, 1.0, _sky_weights.w) if stars_enabled else 0.0
	_sky_material.set_shader_parameter("star_visibility", star_amount)
	_sky_material.set_shader_parameter("star_density", stars_density)
	_sky_material.set_shader_parameter("star_brightness", stars_brightness)
	_sky_material.set_shader_parameter("stars_over_panorama", stars_over_panorama)

	_sky_material.set_shader_parameter("tex_weights", _sky_weights)
	_sky_material.set_shader_parameter("panorama_strength", panorama_strength)
	_sky_material.set_shader_parameter("panorama_brightness", panorama_brightness)
	environment.sky_rotation = Vector3(0.0, deg_to_rad(sky_rotation_degrees), 0.0)


# Веса частей суток для смешивания панорам: x - утро, y - день, z - вечер, w - ночь.
func _weights_for(from_id: int, to_id: int, t: float) -> Vector4:
	var w: Array = [0.0, 0.0, 0.0, 0.0]
	w[from_id] = float(w[from_id]) + (1.0 - t)
	w[to_id] = float(w[to_id]) + t
	return Vector4(w[1], w[2], w[3], w[0])


# Clear Color нельзя менять по времени суток, поэтому переключаем на Color с тем же цветом.
func _prepare_background() -> void:
	var env: Environment = environment
	if control_sky_color and env.background_mode == Environment.BG_CLEAR_COLOR:
		env.background_mode = Environment.BG_COLOR
		env.background_color = _night.background_color


# --- Спрайты солнца и луны ------------------------------------------------------

func _create_sky_sprite(tex: Texture2D, size_degrees: float) -> Sprite3D:
	var sprite := Sprite3D.new()
	sprite.texture = tex
	sprite.billboard = BaseMaterial3D.BILLBOARD_ENABLED
	sprite.shaded = false
	sprite.transparent = true
	sprite.double_sided = true
	sprite.cast_shadow = GeometryInstance3D.SHADOW_CASTING_SETTING_OFF
	if pixel_style:
		sprite.texture_filter = BaseMaterial3D.TEXTURE_FILTER_NEAREST
	else:
		sprite.texture_filter = BaseMaterial3D.TEXTURE_FILTER_LINEAR_WITH_MIPMAPS
	# Подбираем размер в метрах так, чтобы на небе спрайт выглядел нужного углового размера.
	var world_size: float = 2.0 * sky_distance * tan(deg_to_rad(size_degrees) * 0.5)
	sprite.pixel_size = world_size / maxf(float(tex.get_width()), 1.0)
	sprite.visible = false
	add_child(sprite)
	return sprite


# Солнце встаёт с одной стороны, проходит по небу и садится с другой. Луна всегда напротив солнца:
# она восходит там, где солнце село, и заходит там, где оно встаёт.
func _update_sky_bodies() -> void:
	if _sun_sprite == null and _moon_sprite == null:
		return
	var camera: Camera3D = get_viewport().get_camera_3d()
	if camera == null:
		return

	var elevation: float = _sun_elevation()
	var yaw: float = _sun_yaw()

	if _sun_sprite != null:
		var sun_dir: Vector3 = Basis.from_euler(Vector3(deg_to_rad(-elevation), deg_to_rad(yaw), 0.0)).z.normalized()
		var sun_alpha: float = smoothstep(-3.0, 4.0, elevation)
		_sun_sprite.visible = sun_alpha > 0.01
		_sun_sprite.global_position = camera.global_position + sun_dir * sky_distance
		# Низко над горизонтом солнце краснеет.
		var warm: float = smoothstep(2.0, 25.0, elevation)
		var sun_color: Color = sunset_tint.lerp(sun_tint, warm)
		_sun_sprite.modulate = Color(sun_color.r, sun_color.g, sun_color.b, sun_alpha)

	if _moon_sprite != null:
		# Луна: высота зеркальна солнцу, направление по горизонту развёрнуто на 180 градусов.
		var moon_dir: Vector3 = Basis.from_euler(Vector3(deg_to_rad(elevation), deg_to_rad(yaw + 180.0), 0.0)).z.normalized()
		var moon_alpha: float = smoothstep(-3.0, 4.0, -elevation)
		_moon_sprite.visible = moon_alpha > 0.01
		_moon_sprite.global_position = camera.global_position + moon_dir * sky_distance
		_moon_sprite.modulate = Color(moon_tint.r, moon_tint.g, moon_tint.b, moon_alpha)


func _get_procedural_sky() -> ProceduralSkyMaterial:
	var env: Environment = environment
	if env == null or env.sky == null:
		return null
	return env.sky.sky_material as ProceduralSkyMaterial


# Запоминаем текущие настройки WorldEnvironment как "ночь".
func _capture_current_as_preset() -> TimeOfDayPreset:
	var env: Environment = environment
	var p := TimeOfDayPreset.new()
	p.ambient_color = env.ambient_light_color
	p.ambient_energy = env.ambient_light_energy
	p.background_energy = env.background_energy_multiplier
	if env.background_mode == Environment.BG_CLEAR_COLOR:
		# Цвет фона берётся из настроек проекта (Rendering > Environment > Default Clear Color).
		p.background_color = ProjectSettings.get_setting("rendering/environment/defaults/default_clear_color", Color.BLACK)
	else:
		p.background_color = env.background_color
	p.fog_color = env.fog_light_color
	p.fog_density = env.fog_density
	p.exposure = env.tonemap_exposure
	var sky_material := _get_procedural_sky()
	if sky_material != null:
		p.sky_top_color = sky_material.sky_top_color
		p.sky_horizon_color = sky_material.sky_horizon_color
		p.ground_color = sky_material.ground_bottom_color
	else:
		# Берём цвет фона (при Clear Color это цвет из настроек проекта), он уже записан выше.
		p.sky_top_color = p.background_color
		p.sky_horizon_color = p.background_color
		p.ground_color = p.background_color
	if sun_light != null:
		p.sun_color = sun_light.light_color
		p.sun_energy = sun_light.light_energy
	return p


func _make_default(kind: StringName) -> TimeOfDayPreset:
	var p := TimeOfDayPreset.new()
	match kind:
		&"morning":
			p.ambient_color = Color(0.75, 0.70, 0.65)
			p.ambient_energy = 0.8
			p.background_color = Color(0.85, 0.75, 0.65)
			p.sky_top_color = Color(0.45, 0.60, 0.85)
			p.sky_horizon_color = Color(0.95, 0.80, 0.65)
			p.ground_color = Color(0.40, 0.38, 0.35)
			p.fog_color = Color(0.85, 0.78, 0.70)
			p.fog_density = 0.004
			p.exposure = 1.0
			p.sun_color = Color(1.0, 0.85, 0.70)
			p.sun_energy = 0.8
		&"noon":
			p.ambient_color = Color(0.85, 0.88, 0.95)
			p.ambient_energy = 1.0
			p.background_color = Color(0.55, 0.75, 0.95)
			p.sky_top_color = Color(0.30, 0.55, 0.95)
			p.sky_horizon_color = Color(0.70, 0.82, 0.95)
			p.ground_color = Color(0.45, 0.42, 0.38)
			p.fog_color = Color(0.80, 0.85, 0.90)
			p.fog_density = 0.002
			p.exposure = 1.0
			p.sun_color = Color(1.0, 0.97, 0.90)
			p.sun_energy = 1.3
		&"evening":
			p.ambient_color = Color(0.70, 0.50, 0.45)
			p.ambient_energy = 0.7
			p.background_color = Color(0.75, 0.45, 0.35)
			p.sky_top_color = Color(0.25, 0.30, 0.55)
			p.sky_horizon_color = Color(0.95, 0.50, 0.30)
			p.ground_color = Color(0.30, 0.25, 0.25)
			p.fog_color = Color(0.80, 0.50, 0.40)
			p.fog_density = 0.005
			p.exposure = 1.0
			p.sun_color = Color(1.0, 0.55, 0.30)
			p.sun_energy = 0.9
	return p
