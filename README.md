# Segfault in vkCreateRayTracingPipelinesKHR

I've been porting vk_mini_path_tracer. At [e11_rt_pipeline2](https://github.com/nvpro-samples/vk_mini_path_tracer/tree/main/checkpoints/e11_rt_pipeline_2) my validated SPIR-V module segfaults the driver inside `vkCreateRayTracingPipelinesKHR`.

```
Program received signal SIGSEGV, Segmentation fault.
0x00007fffed6b74d8 in ?? () from /usr/lib/x86_64-linux-gnu/libnvidia-glvkspirv.so.455.46.04
(gdb) where
#0  0x00007fffed6b74d8 in ?? () from /usr/lib/x86_64-linux-gnu/libnvidia-glvkspirv.so.455.46.04
#1  0x00007fffed574fb2 in ?? () from /usr/lib/x86_64-linux-gnu/libnvidia-glvkspirv.so.455.46.04
#2  0x00007fffed579248 in ?? () from /usr/lib/x86_64-linux-gnu/libnvidia-glvkspirv.so.455.46.04
#3  0x00007fffed6c816f in ?? () from /usr/lib/x86_64-linux-gnu/libnvidia-glvkspirv.so.455.46.04
#4  0x00007fffed585e92 in ?? () from /usr/lib/x86_64-linux-gnu/libnvidia-glvkspirv.so.455.46.04
#5  0x00007fffed58003d in _nv002nvvm () from /usr/lib/x86_64-linux-gnu/libnvidia-glvkspirv.so.455.46.04
#6  0x00007ffff668879d in ?? () from /usr/lib/x86_64-linux-gnu/libnvidia-glcore.so.455.46.04
#7  0x00007ffff6688a10 in ?? () from /usr/lib/x86_64-linux-gnu/libnvidia-glcore.so.455.46.04
#8  0x00007ffff669812e in ?? () from /usr/lib/x86_64-linux-gnu/libnvidia-glcore.so.455.46.04
#9  0x00007ffff6698245 in ?? () from /usr/lib/x86_64-linux-gnu/libnvidia-glcore.so.455.46.04
#10 0x00007ffff6698409 in ?? () from /usr/lib/x86_64-linux-gnu/libnvidia-glcore.so.455.46.04
#11 0x00007fffeea96128 in ?? () from /usr/lib/x86_64-linux-gnu/libVkLayer_khronos_validation.so
#12 0x00007fffeea00b19 in ?? () from /usr/lib/x86_64-linux-gnu/libVkLayer_khronos_validation.so
#13 0x00000000004ed43e in vkCreateRayTracingPipelinesKHR (device=0xb5aa88, deferredOperation=0x0, pipeline
Cache=0x0, createInfoCount=1, pCreateInfos=0x7fffffffbdb8, pAllocator=0x0, pPipelines=0x7fffffffc018) at e
xtensions_vk.cpp:762
#14 0x0000000000498ef8 in main (argc=1, argv=0x7fffffffdfc8) at main.cpp:761
```

Driver Version: 455.46.04 
Linux Mint 20 / Ubuntu 20.04.

Slot in the included main.cpp into the nvpro samples framework to build.

## Update

I was able to segfault with the smallest possible pipeline that uses `OpTraceRayKHR`. See tiny.spv.

```
               OpCapability Shader
               OpCapability RayTracingKHR
               OpExtension "GL_EXT_scalar_block_layout"
               OpExtension "SPV_KHR_ray_tracing"
               OpMemoryModel Logical GLSL450
               OpEntryPoint RayGenerationNV %_Z11rgen_shaderv "_Z11rgen_shaderv" %shader_tlas
               OpEntryPoint MissNV %_Z12rmiss_shaderv "_Z12rmiss_shaderv"
               OpEntryPoint ClosestHitNV %_Z12rchit_shaderI11material0_tEvv "_Z12rchit_shaderI11material0_tEvv"
               OpName %shader_tlas "shader_tlas"
               OpName %bb_entry "bb-entry"
               OpName %_Z11rgen_shaderv "_Z11rgen_shaderv"
               OpName %bb_entry_0 "bb-entry"
               OpName %_Z12rmiss_shaderv "_Z12rmiss_shaderv"
               OpName %bb_entry_1 "bb-entry"
               OpName %_Z12rchit_shaderI11material0_tEvv "_Z12rchit_shaderI11material0_tEvv"
               OpDecorate %shader_tlas Binding 1
               OpDecorate %shader_tlas DescriptorSet 0
          %2 = OpTypeAccelerationStructureKHR
%_ptr_UniformConstant_2 = OpTypePointer UniformConstant %2
%shader_tlas = OpVariable %_ptr_UniformConstant_2 UniformConstant
       %void = OpTypeVoid
          %8 = OpTypeFunction %void
       %uint = OpTypeInt 32 0
     %uint_1 = OpConstant %uint 1
   %uint_255 = OpConstant %uint 255
     %uint_0 = OpConstant %uint 0
      %float = OpTypeFloat 32
    %v3float = OpTypeVector %float 3
    %float_0 = OpConstant %float 0
         %18 = OpConstantComposite %v3float %float_0 %float_0 %float_0
    %float_1 = OpConstant %float 1
         %20 = OpConstantComposite %v3float %float_1 %float_0 %float_0
%float_10000 = OpConstant %float 10000
        %int = OpTypeInt 32 1
      %int_0 = OpConstant %int 0
%_Z11rgen_shaderv = OpFunction %void None %8
   %bb_entry = OpLabel
         %10 = OpLoad %2 %shader_tlas
               OpTraceRayKHR %10 %uint_1 %uint_255 %uint_0 %uint_0 %uint_0 %18 %float_0 %20 %float_10000 %int_0
               OpReturn
               OpFunctionEnd
%_Z12rmiss_shaderv = OpFunction %void None %8
 %bb_entry_0 = OpLabel
               OpReturn
               OpFunctionEnd
%_Z12rchit_shaderI11material0_tEvv = OpFunction %void None %8
 %bb_entry_1 = OpLabel
               OpReturn
               OpFunctionEnd
```

## Validator output

I'm running with the Vulkan validation layers and they don't flag anything. Here is the full output:

```_______________
Vulkan Version:
 - available:  1.2.154
 - requesting: 1.2.0
___________________________
Available Instance Layers :
VK_LAYER_NV_optimus (v. 1.2.164 1) : NVIDIA Optimus layer
VK_LAYER_NV_optimus (v. 1.2.133 1) : NVIDIA Optimus layer
VK_LAYER_LUNARG_gfxreconstruct (v. 1.2.154 9003) : GFXReconstruct Capture Layer Version 0.9.3-unknown
VK_LAYER_LUNARG_api_dump (v. 1.2.162 2) : LunarG API dump layer
VK_LAYER_LUNARG_device_simulation (v. 1.2.162 1) : LunarG device simulation layer
VK_LAYER_MESA_overlay (v. 1.1.73 1) : Mesa Overlay layer
VK_LAYER_KHRONOS_validation (v. 1.2.162 1) : Khronos Validation Layer
VK_LAYER_LUNARG_monitor (v. 1.2.162 1) : Execution Monitoring Layer
VK_LAYER_LUNARG_screenshot (v. 1.2.162 1) : LunarG image capture layer

Available Instance Extensions :
VK_KHR_device_group_creation (v. 1)
VK_KHR_display (v. 23)
VK_KHR_external_fence_capabilities (v. 1)
VK_KHR_external_memory_capabilities (v. 1)
VK_KHR_external_semaphore_capabilities (v. 1)
VK_KHR_get_display_properties2 (v. 1)
VK_KHR_get_physical_device_properties2 (v. 2)
VK_KHR_get_surface_capabilities2 (v. 1)
VK_KHR_surface (v. 25)
VK_KHR_surface_protected_capabilities (v. 1)
VK_KHR_xcb_surface (v. 6)
VK_KHR_xlib_surface (v. 6)
VK_EXT_acquire_xlib_display (v. 1)
VK_EXT_debug_report (v. 9)
VK_EXT_debug_utils (v. 2)
VK_EXT_direct_mode_display (v. 1)
VK_EXT_display_surface_counter (v. 1)
VK_KHR_wayland_surface (v. 6)
______________________
Used Instance Layers :
VK_LAYER_KHRONOS_validation

Used Instance Extensions :
VK_EXT_debug_utils
____________________
Compatible Devices :
0: GeForce RTX 2060
1: GeForce RTX 2060
Physical devices found : 2
_____________________________
Available Device Extensions :
VK_KHR_16bit_storage (v. 1)
VK_KHR_8bit_storage (v. 1)
VK_KHR_acceleration_structure (v. 11)
VK_KHR_bind_memory2 (v. 1)
VK_KHR_buffer_device_address (v. 1)
VK_KHR_copy_commands2 (v. 1)
VK_KHR_create_renderpass2 (v. 1)
VK_KHR_dedicated_allocation (v. 3)
VK_KHR_deferred_host_operations (v. 4)
VK_KHR_depth_stencil_resolve (v. 1)
VK_KHR_descriptor_update_template (v. 1)
VK_KHR_device_group (v. 4)
VK_KHR_draw_indirect_count (v. 1)
VK_KHR_driver_properties (v. 1)
VK_KHR_external_fence (v. 1)
VK_KHR_external_fence_fd (v. 1)
VK_KHR_external_memory (v. 1)
VK_KHR_external_memory_fd (v. 1)
VK_KHR_external_semaphore (v. 1)
VK_KHR_external_semaphore_fd (v. 1)
VK_KHR_fragment_shading_rate (v. 1)
VK_KHR_get_memory_requirements2 (v. 1)
VK_KHR_image_format_list (v. 1)
VK_KHR_imageless_framebuffer (v. 1)
VK_KHR_maintenance1 (v. 2)
VK_KHR_maintenance2 (v. 1)
VK_KHR_maintenance3 (v. 1)
VK_KHR_multiview (v. 1)
VK_KHR_pipeline_executable_properties (v. 1)
VK_KHR_pipeline_library (v. 1)
VK_KHR_push_descriptor (v. 2)
VK_KHR_ray_query (v. 1)
VK_KHR_ray_tracing_pipeline (v. 1)
VK_KHR_relaxed_block_layout (v. 1)
VK_KHR_sampler_mirror_clamp_to_edge (v. 3)
VK_KHR_sampler_ycbcr_conversion (v. 14)
VK_KHR_separate_depth_stencil_layouts (v. 1)
VK_KHR_shader_atomic_int64 (v. 1)
VK_KHR_shader_clock (v. 1)
VK_KHR_shader_draw_parameters (v. 1)
VK_KHR_shader_float16_int8 (v. 1)
VK_KHR_shader_float_controls (v. 4)
VK_KHR_shader_non_semantic_info (v. 1)
VK_KHR_shader_subgroup_extended_types (v. 1)
VK_KHR_shader_terminate_invocation (v. 1)
VK_KHR_spirv_1_4 (v. 1)
VK_KHR_storage_buffer_storage_class (v. 1)
VK_KHR_swapchain (v. 70)
VK_KHR_swapchain_mutable_format (v. 1)
VK_KHR_timeline_semaphore (v. 2)
VK_KHR_uniform_buffer_standard_layout (v. 1)
VK_KHR_variable_pointers (v. 1)
VK_KHR_vulkan_memory_model (v. 3)
VK_EXT_4444_formats (v. 1)
VK_EXT_blend_operation_advanced (v. 2)
VK_EXT_buffer_device_address (v. 2)
VK_EXT_calibrated_timestamps (v. 1)
VK_EXT_conditional_rendering (v. 2)
VK_EXT_conservative_rasterization (v. 1)
VK_EXT_custom_border_color (v. 12)
VK_EXT_depth_clip_enable (v. 1)
VK_EXT_depth_range_unrestricted (v. 1)
VK_EXT_descriptor_indexing (v. 2)
VK_EXT_discard_rectangles (v. 1)
VK_EXT_display_control (v. 1)
VK_EXT_extended_dynamic_state (v. 1)
VK_EXT_external_memory_host (v. 1)
VK_EXT_fragment_shader_interlock (v. 1)
VK_EXT_global_priority (v. 2)
VK_EXT_host_query_reset (v. 1)
VK_EXT_image_robustness (v. 1)
VK_EXT_index_type_uint8 (v. 1)
VK_EXT_inline_uniform_block (v. 1)
VK_EXT_line_rasterization (v. 1)
VK_EXT_memory_budget (v. 1)
VK_EXT_pci_bus_info (v. 2)
VK_EXT_pipeline_creation_cache_control (v. 3)
VK_EXT_pipeline_creation_feedback (v. 1)
VK_EXT_post_depth_coverage (v. 1)
VK_EXT_private_data (v. 1)
VK_EXT_robustness2 (v. 1)
VK_EXT_sample_locations (v. 1)
VK_EXT_sampler_filter_minmax (v. 2)
VK_EXT_scalar_block_layout (v. 1)
VK_EXT_separate_stencil_usage (v. 1)
VK_EXT_shader_atomic_float (v. 1)
VK_EXT_shader_demote_to_helper_invocation (v. 1)
VK_EXT_shader_image_atomic_int64 (v. 1)
VK_EXT_shader_subgroup_ballot (v. 1)
VK_EXT_shader_subgroup_vote (v. 1)
VK_EXT_shader_viewport_index_layer (v. 1)
VK_EXT_subgroup_size_control (v. 2)
VK_EXT_texel_buffer_alignment (v. 1)
VK_EXT_tooling_info (v. 1)
VK_EXT_transform_feedback (v. 1)
VK_EXT_vertex_attribute_divisor (v. 3)
VK_EXT_ycbcr_image_arrays (v. 1)
VK_NV_clip_space_w_scaling (v. 1)
VK_NV_compute_shader_derivatives (v. 1)
VK_NV_cooperative_matrix (v. 1)
VK_NV_corner_sampled_image (v. 2)
VK_NV_coverage_reduction_mode (v. 1)
VK_NV_cuda_kernel_launch (v. 2)
VK_NV_dedicated_allocation (v. 1)
VK_NV_dedicated_allocation_image_aliasing (v. 1)
VK_NV_device_diagnostic_checkpoints (v. 2)
VK_NV_device_diagnostics_config (v. 1)
VK_NV_device_generated_commands (v. 3)
VK_NV_fill_rectangle (v. 1)
VK_NV_fragment_coverage_to_color (v. 1)
VK_NV_fragment_shader_barycentric (v. 1)
VK_NV_fragment_shading_rate_enums (v. 1)
VK_NV_framebuffer_mixed_samples (v. 1)
VK_NV_geometry_shader_passthrough (v. 1)
VK_NV_mesh_shader (v. 1)
VK_NV_ray_tracing (v. 3)
VK_NV_representative_fragment_test (v. 2)
VK_NV_sample_mask_override_coverage (v. 1)
VK_NV_scissor_exclusive (v. 1)
VK_NV_shader_image_footprint (v. 2)
VK_NV_shader_sm_builtins (v. 1)
VK_NV_shader_subgroup_partitioned (v. 1)
VK_NV_shading_rate_image (v. 3)
VK_NV_viewport_array2 (v. 1)
VK_NV_viewport_swizzle (v. 1)
VK_NVX_binary_import (v. 1)
VK_NVX_image_view_handle (v. 2)
VK_NVX_multiview_per_view_attributes (v. 1)
________________________
Used Device Extensions :
VK_KHR_deferred_host_operations
VK_KHR_acceleration_structure
VK_KHR_ray_tracing_pipeline

INFO: Loader Message 
 --> Inserted device layer VK_LAYER_KHRONOS_validation (libVkLayer_khronos_validation.so)
 RT BLAS: reducing from: 6912 to: 2048 = 4864 (70.37% smaller) 
Loaded file segfault.spv with 14252 bytes
Segmentation fault (core dumped)
```
