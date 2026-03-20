import os
import shutil

# ==============================
# CONFIGURACIÓN
# ==============================
GAME_ID = "ULUS10537"

ISO_EXTRAIDO = "C:/DBZ_ISO_EDIT/"   # juego base extraído
MOD_PATH = "C:/MOD_DBZ_SUPER/"      # tu mod
BUILD_PATH = "C:/DBZ_BUILD_FINAL/"  # salida final

MAX_TOTAL_SIZE = 1.7 * 1024 * 1024 * 1024  # 1.7 GB
MAX_FILE_SIZE = 2 * 1024 * 1024  # 2MB por archivo

FORMATOS_VALIDOS = (".png", ".dds", ".at3")

# ==============================
# LIMPIAR BUILD
# ==============================
if os.path.exists(BUILD_PATH):
    shutil.rmtree(BUILD_PATH)

shutil.copytree(ISO_EXTRAIDO, BUILD_PATH)

print("✔ Base del juego copiada")

# ==============================
# COPIAR MOD OPTIMIZADO
# ==============================
def copiar_mod(origen, destino):
    for root, dirs, files in os.walk(origen):
        for file in files:
            if not file.lower().endswith(FORMATOS_VALIDOS):
                continue

            ruta_origen = os.path.join(root, file)
            ruta_destino = os.path.join(destino, file)

            try:
                size = os.path.getsize(ruta_origen)

                if size > MAX_FILE_SIZE:
                    print(f"⚠ Omitido por tamaño: {file}")
                    continue

                os.makedirs(os.path.dirname(ruta_destino), exist_ok=True)
                shutil.copy2(ruta_origen, ruta_destino)

                print(f"✔ Copiado: {file}")

            except Exception as e:
                print(f"❌ Error: {file} -> {e}")

copiar_mod(MOD_PATH, BUILD_PATH)

# ==============================
# CALCULAR PESO TOTAL
# ==============================
def calcular_peso(ruta):
    total = 0
    for root, dirs, files in os.walk(ruta):
        for file in files:
            total += os.path.getsize(os.path.join(root, file))
    return total

peso_total = calcular_peso(BUILD_PATH)

print(f"\n📦 Peso actual: {round(peso_total / (1024**3), 2)} GB")

# ==============================
# OPTIMIZAR SI EXCEDE
# ==============================
if peso_total > MAX_TOTAL_SIZE:
    print("⚠ Excede 1.7GB, optimizando...")

    for root, dirs, files in os.walk(BUILD_PATH):
        for file in files:
            ruta = os.path.join(root, file)

            try:
                if os.path.getsize(ruta) > MAX_FILE_SIZE:
                    os.remove(ruta)
                    print(f"🗑 Eliminado: {file}")

            except:
                pass

    peso_total = calcular_peso(BUILD_PATH)

# ==============================
# RESULTADO FINAL
# ==============================
print("\n==============================")
print("✔ BUILD COMPLETADO")
print(f"📦 Peso final: {round(peso_total / (1024**3), 2)} GB")

if peso_total <= MAX_TOTAL_SIZE:
    print("✅ Listo para crear ISO en UMDGen")
else:
    print("❌ Aún excede tamaño, reduce texturas")
