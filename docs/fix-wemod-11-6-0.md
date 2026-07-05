# WeMod 11.6.0 — Atualização do ActivatePro

## Problema

No WeMod 11.6.0, o patch ActivatePro falhava com:

```
[ERROR] Failed to patch: [ENHANCER] Failed to apply patches: ActivatePro. The version may not be supported.
```

## Causas

### 1. Método removido: `setAccountWandBrandExperience`

O WeMod 11.6.0 removeu o método `setAccountWandBrandExperience()` da service class de account. O sub-patch que buscava esse método nunca encontrava match, mas o Wand Enhancer exige que **todos** os sub-patches de um grupo sejam aplicados — caso contrário, lança exceção no final.

**Solução:** Remover o PatchEntry `setAccountWandBrandExperience` de `EnhancerConfig.cs`. O método não existe mais no bundle JS do WeMod 11.6.0.

### 2. Overlay bundles ignorados

O `IsCandidateBundleFile()` em `Enhancer.cs` só aceitava arquivos com padrão `app-*.bundle.js` e `index.js`. O WeMod 11.6.0 possui bundles separados para a janela overlay (`overlay-*.bundle.js`) que contêm o mesmo reducer `ACTION_SET_ACCOUNT`. Como esses arquivos nunca eram varridos, o reducer do overlay ficava sem patch.

**Solução:** Adicionar `overlay-*.bundle.js` como candidato em `IsCandidateBundleFile()`.

## Arquivos modificados

### `WandEnhancer/Core/Enhancer.cs`

- Adicionada constante `OverlayBundleFilePrefix = "overlay-"`
- `IsCandidateBundleFile()` agora também aceita arquivos começando com `overlay-` e terminando com `.bundle.js`

### `WandEnhancer/Core/EnhancerConfig.cs`

- Removido o PatchEntry `setAccountWandBrandExperience` (método não existe no WeMod 11.6.0)

## Verificação

Com a correção, os 4 sub-patches do ActivatePro são aplicados com sucesso:

```
[ActivatePro -> getUserAccount]         → app-4e8d88bd.*.bundle.js
[ActivatePro -> setAccountLanguage]     → app-4e8d88bd.*.bundle.js
[ActivatePro -> disableNativeRemotePairing] → app-4e8d88bd.*.bundle.js
[ActivatePro -> setAccountReducer]      → app-6f178a3f.*.bundle.js
                                        → overlay-6f178a3f.*.bundle.js ✓ (novo)
```

## Compatibilidade

Testado com:
- WeMod 11.6.0 (Electron 34)
- Wand Enhancer 1.0.9.3
- GE-Proton11-1
