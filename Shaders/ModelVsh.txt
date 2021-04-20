#ifdef HLSL

// Transform
float4x4 u_worldMatrix[MAX_INSTANCES_COUNT];
float4x4 u_worldViewProjectionMatrix[MAX_INSTANCES_COUNT];

// Material
float4 u_materialColor;

// Emission color
float4 u_emissionColor;

// Light
float3 u_ambientLightColor;
float3 u_diffuseLightColor1;
float3 u_directionToLight1;
float3 u_diffuseLightColor2;
float3 u_directionToLight2;

// Fog
float2 u_fogStartInvLength;
float u_fogYMultiplier;
float3 u_worldUp;

void main(
	in float3 a_position: POSITION,
	in float3 a_normal: NORMAL,
	in float2 a_texcoord: TEXCOORD,
	in float a_instance: INSTANCE,
	out float4 v_color : COLOR,
	out float2 v_texcoord : TEXCOORD,
	out float v_fog : FOG,
	out float4 sv_position: SV_POSITION
)
{
	// Texture
	v_texcoord = a_texcoord;
	
	// Instancing
	int instance = int(a_instance);

	// Normal
	float3 worldNormal = normalize(mul(float4(a_normal, 0.0), u_worldMatrix[instance]).xyz);

	// Lighting
	v_color = u_materialColor;
	float3 lightColor = u_ambientLightColor;
	lightColor += u_diffuseLightColor1 * max(dot(u_directionToLight1, worldNormal), 0.0);
	lightColor += u_diffuseLightColor2 * max(dot(u_directionToLight2, worldNormal), 0.0);
	v_color = float4(lightColor, 1) * u_materialColor;

	// Emission color
	v_color += u_emissionColor;
	
	// Fog
	float3 worldPosition = mul(float4(a_position, 1.0), u_worldMatrix[instance]).xyz;
	float worldY = dot(worldPosition, u_worldUp);
	float d = sqrt((u_fogYMultiplier * u_fogYMultiplier - 1) * (worldY * worldY) + dot(worldPosition, worldPosition));	// Reduce Y component of worldPosition in distance calculation
	v_fog = saturate((d - u_fogStartInvLength.x) * u_fogStartInvLength.y);

	// Position
	sv_position = mul(float4(a_position, 1.0), u_worldViewProjectionMatrix[instance]);
}

#endif
#ifdef GLSL

// <Semantic Name='POSITION' Attribute='a_position' />
// <Semantic Name='NORMAL' Attribute='a_normal' />
// <Semantic Name='COLOR' Attribute='a_color' />
// <Semantic Name='TEXCOORD' Attribute='a_texcoord' />
// <Semantic Name='INSTANCE' Attribute='a_instance' />

// Transform
uniform mat4 u_worldMatrix[MAX_INSTANCES_COUNT];
uniform mat4 u_worldViewProjectionMatrix[MAX_INSTANCES_COUNT];

uniform vec4 u_materialColor;

// Emission color
uniform vec4 u_emissionColor;

// Material and light
uniform vec3 u_ambientLightColor;
uniform vec3 u_diffuseLightColor1;
uniform vec3 u_directionToLight1;
uniform vec3 u_diffuseLightColor2;
uniform vec3 u_directionToLight2;

// Fog
uniform vec2 u_fogStartInvLength;
uniform float u_fogYMultiplier;
uniform vec3 u_worldUp;

// Inputs and outputs
attribute vec3 a_position;
attribute vec3 a_normal;
attribute vec2 a_texcoord;
attribute float a_instance;
varying vec4 v_color;
varying vec2 v_texcoord;
varying float v_fog;

void main()
{
	// Texture
	v_texcoord = a_texcoord;
	
	// Instancing
	int instance = int(a_instance);
	
	// Normal
	vec3 worldNormal = normalize(vec3(u_worldMatrix[instance] * vec4(a_normal, 0.0)));

	// Lighting
	v_color = u_materialColor;
	vec3 lightColor = u_ambientLightColor;
	lightColor += u_diffuseLightColor1 * max(dot(u_directionToLight1, worldNormal), 0.0);
	lightColor += u_diffuseLightColor2 * max(dot(u_directionToLight2, worldNormal), 0.0);
	v_color = vec4(lightColor, 1) * u_materialColor;

	// Emission color
	v_color += u_emissionColor;

	// Fog
	vec3 worldPosition = vec3(u_worldMatrix[instance] * vec4(a_position, 1.0));
	float worldY = dot(worldPosition, u_worldUp);
	float d = sqrt((u_fogYMultiplier * u_fogYMultiplier - 1.0) * (worldY * worldY) + dot(worldPosition, worldPosition));
	v_fog = clamp((d - u_fogStartInvLength.x) * u_fogStartInvLength.y, 0.0, 1.0);
	
	// Position
	gl_Position = u_worldViewProjectionMatrix[instance] * vec4(a_position, 1.0);

	// Fix gl_Position
	OPENGL_POSITION_FIX;
}

#endif
