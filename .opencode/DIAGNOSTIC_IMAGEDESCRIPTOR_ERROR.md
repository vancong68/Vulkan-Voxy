# ImageDescriptor NoSuchMethodError - Diagnostic Report

## Current Repository State

**Current commit:** `5fffd509 - fix imagedescriptorbug`  
**VulkanMod version in use:** 0.6.5 (Modrinth ID: zLXOtN34)  
**Minecraft version:** 26.1.2

## ImageDescriptor Calls in Codebase

**Found:** 2 calls total (both in VulkanBerylSectionDrawPipeline.java, lines 6556-6557)

```java
new ImageDescriptor(7, "sampler2D", "blockModelAtlas", VTextureSelector.getTextureIdx("Sampler0"), VK10.VK_DESCRIPTOR_TYPE_COMBINED_IMAGE_SAMPLER)
new ImageDescriptor(8, "sampler2D", "depthTex", VTextureSelector.getTextureIdx("Sampler7"), VK10.VK_DESCRIPTOR_TYPE_COMBINED_IMAGE_SAMPLER)
```

**Parameter count:** 5 parameters (each call)  
**Type:** Binding (int), Type string ("sampler2D"), Name (string), ImageIdx (int), DescriptorType (int)

## VulkanMod 0.6.5 - ImageDescriptor Constructors

**All three overloads exist in 0.6.5:**

```java
public ImageDescriptor(int binding, String type, String name, int imageIdx)
public ImageDescriptor(int binding, String type, String name, int imageIdx, boolean isStorageImage)
public ImageDescriptor(int binding, String type, String name, int imageIdx, int descriptorType)
```

- 4-parameter: ✅ EXISTS
- 5-parameter (boolean): ✅ EXISTS  
- 5-parameter (int): ✅ EXISTS

**Conclusion:** Both old 4-parameter AND new 5-parameter constructors are available at runtime.

## Compilation Status

- ✅ `./gradlew clean compileJava` - **BUILD SUCCESSFUL**
- ✅ No compilation errors related to ImageDescriptor
- ✅ No constructor-related warnings

## Possible Reasons for NoSuchMethodError

If you're still getting `java.lang.NoSuchMethodError: ImageDescriptor.<init>(int,String,String,int)` AFTER this commit:

### Reason 1: Stale JAR in Mod Folder ⚠️ **MOST LIKELY**

**Symptom:** Error mentions 4-parameter constructor, but code has 5-parameter

**Solution:**
1. Stop Minecraft completely
2. Delete old JAR from `~/.local/share/PrismLauncher/instances/[instance]/minecraft/mods/`  
3. Rebuild: `./gradlew build`
4. Copy fresh JAR from `build/libs/` to mods folder
5. Restart game

### Reason 2: Cached Build Artifacts

**Symptom:** Compilation appears to work, but runtime fails

**Solution:**
```bash
cd Vulkan-Voxy
./gradlew clean build
```

This guarantees fresh bytecode compilation.

### Reason 3: Wrong VulkanMod Version at Runtime

**Symptom:** Error mentions constructors that don't exist in your version

**Solution:**
1. Verify in PrismLauncher instance that VulkanMod version is correct
2. Check `~/.local/share/PrismLauncher/instances/[instance]/mods/` for VulkanMod JARs
3. Ensure only ONE version of VulkanMod exists
4. Verify it matches build dependencies in `build.gradle` (zLXOtN34)

### Reason 4: Classpath Order Issue

**Symptom:** One mod version loads before another, causing wrong class to be used

**Solution:**
1. In PrismLauncher, delete both VulkanMod and Beryl mods
2. Re-download/reinstall them from Modrinth
3. Ensure proper load order (VulkanMod should load before Vulkan-Voxy)

## Verification Steps

### Step 1: Confirm Source Code

```bash
grep -n "new ImageDescriptor" src/main/java/me/cortex/voxy/client/core/rendering/section/backend/vulkanberyl/VulkanBerylSectionDrawPipeline.java
```

Expected output:
```
6556: ... VK10.VK_DESCRIPTOR_TYPE_COMBINED_IMAGE_SAMPLER),
6557: ... VK10.VK_DESCRIPTOR_TYPE_COMBINED_IMAGE_SAMPLER)
```

✅ Both lines should show **5 parameters** ending with `VK_DESCRIPTOR_TYPE_COMBINED_IMAGE_SAMPLER`

### Step 2: Rebuild JAR

```bash
./gradlew clean build
ls -lh build/libs/
```

Expected: New JAR file with today's timestamp

### Step 3: Deploy JAR

```bash
cp build/libs/vulkan-voxy-*.jar ~/.local/share/PrismLauncher/instances/Vulkan\ Optimized/minecraft/mods/
```

### Step 4: Verify Runtime Mods

In PrismLauncher Launcher:
- Modlist should show:
  - VulkanMod 0.6.5
  - Beryl (latest)
  - Vulkan-Voxy (your newly built version, should have today's build time)

## Current Code Status

✅ **Compilation:** PASSES  
✅ **Constructor calls:** 5-parameter (correct for VulkanMod 0.6.5)  
✅ **VulkanMod 0.6.5:** All required constructors present  

⚠️ **Runtime error still occurring:** Likely caused by stale JAR or wrong mod version loaded

## Explanation of the Error

When you see:
```
java.lang.NoSuchMethodError: ImageDescriptor.<init>(int,String,String,int)
```

This means the JVM searched for **exactly that constructor signature** (4 parameters) and didn't find it.

This can happen if:
1. **Old compiled bytecode** calls 4-param constructor
2. **Wrong VulkanMod version** at runtime doesn't have 4-param (unlikely with 0.6.5)
3. **Class loading issue** - wrong class file loaded

Since current source code has 5-parameter calls, if you're still seeing 4-parameter error, the bytecode being executed is **NOT from the current source**.

## Recommended Next Steps

1. **Clean rebuild:**
   ```bash
   ./gradlew clean build
   ```

2. **Verify JAR contents** (if possible):
   ```bash
   jar tf build/libs/vulkan-voxy-*.jar | grep VulkanBerylSectionDrawPipeline
   ```

3. **Stop and restart game** completely (not just reload)

4. **Check game log** for exact error location and stack trace

5. **If error persists:** Share the full stack trace including:
   - Which mod is loading VulkanMod
   - Exact error message with line number
   - PrismLauncher instance mods list

## Summary

| Item | Status |
|------|--------|
| Source code ImageDescriptor calls | ✅ 5-parameter (current) |
| Compilation | ✅ Successful |
| VulkanMod 0.6.5 constructors | ✅ All present |
| Build cache | ✅ Cleaned |
| JARs | ⚠️ Need fresh rebuild and deploy |

**Most likely solution:** Fresh `./gradlew clean build` + redeploy JAR to mods folder.
