# 【Three.js】VRMモデルとVRMAアニメーションを使ったWebビューアーを作成してみた

## はじめに

VRM（Virtual Reality Model）は、VRアプリケーション向けの人型3Dアバターファイルフォーマットです。この記事では、Three.jsを使ってVRMモデルを表示し、VRMA（VRM Animation）でアニメーションを再生できるWebビューアーを作成した過程と技術的なポイントを紹介します。

## 🎮 デモ

実際に動作するデモはこちらからご覧ください：
**[VRM Viewer Demo](https://TK-256.github.io/vrm-viewer/)**

![VRM Viewer Demo](https://github.com/TK-256/vrm-viewer/blob/main/demo.gif)

## 特徴

- 📱 **レスポンシブデザイン**: デスクトップ・モバイル対応
- 🎭 **VRM 1.0サポート**: VRMモデルの読み込みと表示
- 🎬 **VRMAアニメーション**: カスタムアニメーションファイルの再生
- 🎮 **インタラクティブ操作**: 再生・一時停止・停止制御
- 🎨 **モダンUI**: グラデーションベースの美しいインターフェース
- ⚡ **高速パフォーマンス**: 最適化されたレンダリング

## 使用技術

### 主要ライブラリ

- [Three.js](https://threejs.org/) - 3Dグラフィックスライブラリ
- [@pixiv/three-vrm](https://github.com/pixiv/three-vrm) - VRMモデルサポート
- [@pixiv/three-vrm-animation](https://github.com/pixiv/three-vrm-animation) - VRMAアニメーションサポート

### 技術スタック

- **フロントエンド**: HTML5, CSS3, ES6+ JavaScript
- **3Dレンダリング**: Three.js WebGLRenderer
- **アニメーション**: Three.js AnimationMixer
- **デプロイ**: GitHub Pages

## プロジェクト構成

```
vrm_viewer/
├── index.html              # メインアプリケーション
├── VRM/
│   └── ロング女の子1.vrm     # サンプルVRMモデル
├── VRMA/
│   ├── original_01.vrma    # 頭うなずきアニメーション
│   └── original_02.vrma    # 腕振りアニメーション
├── README.md               # 英語版ドキュメント
└── README-jp.md           # 日本語版ドキュメント
```

## 実装のポイント

### 1. VRMモデルの読み込み

```javascript
// GLTF Loader with VRM Plugin
const loader = new GLTFLoader();
loader.register((parser) => {
    return new VRMLoaderPlugin(parser);
});

async function loadVRM(url) {
    return new Promise((resolve, reject) => {
        loader.load(url, (gltf) => {
            const vrm = gltf.userData.vrm;
            
            // パフォーマンス最適化
            VRMUtils.removeUnnecessaryVertices(gltf.scene);
            VRMUtils.combineSkeletons(gltf.scene);
            VRMUtils.combineMorphs(vrm);
            
            // フラスタムカリングを無効化
            vrm.scene.traverse((obj) => {
                obj.frustumCulled = false;
            });
            
            scene.add(vrm.scene);
            vrm.scene.rotation.y = Math.PI; // モデルを正面に回転
            currentVrm = vrm;
            
            resolve(vrm);
        });
    });
}
```

### 2. VRMAアニメーションの処理

```javascript
// VRMA Animation Plugin
loader.register((parser) => {
    return new VRMAnimationLoaderPlugin(parser);
});

async function loadVRMA(url) {
    return new Promise((resolve, reject) => {
        loader.load(url, (gltf) => {
            const vrmAnimationData = gltf.userData.vrmAnimations && gltf.userData.vrmAnimations[0];
            
            if (vrmAnimationData) {
                // VRMアニメーションからThree.js AnimationClipを生成
                const clip = createVRMAnimationClip(vrmAnimationData, currentVrm);
                vrmaAnimationClip = clip;
                resolve(clip);
            }
        });
    });
}
```

### 3. アニメーション制御

```javascript
function playAnimation() {
    if (currentVrm && vrmaAnimationClip && currentMixer) {
        // 既存のアクションを停止
        if (currentAction) {
            currentAction.stop();
        }
        
        // 新しいアニメーションアクションを作成
        currentAction = currentMixer.clipAction(vrmaAnimationClip);
        currentAction.setLoop(THREE.LoopRepeat);
        currentAction.clampWhenFinished = true;
        currentAction.reset();
        currentAction.play();
    }
}
```

### 4. レンダリングループ

```javascript
const clock = new THREE.Clock();

function animate() {
    requestAnimationFrame(animate);
    
    const deltaTime = clock.getDelta();
    
    // VRMとアニメーションミキサーの更新
    if (currentVrm) {
        currentVrm.update(deltaTime);
    }
    if (currentMixer) {
        currentMixer.update(deltaTime);
    }
    
    controls.update();
    renderer.render(scene, camera);
}
```

## VRMAアニメーションの作成

このプロジェクトでは、オリジナルのVRMAアニメーションファイルを2つ作成しました：

### original_01.vrma - 頭うなずきアニメーション
- **動作**: 自然な頭のうなずき動作
- **継続時間**: 6秒
- **技術**: VRMヒューマノイドボーン仕様に準拠した正確なボーンマッピング

### original_02.vrma - 腕振りアニメーション
- **動作**: 左右の腕を交互に振る動作
- **継続時間**: 4秒
- **技術**: 上腕と前腕が連動した自然な動き

両アニメーションの特徴：
- ✅ 完全オリジナルで著作権フリー
- ✅ VRM 1.0仕様完全対応
- ✅ 709フレームの高精度アニメーション
- ✅ 91チャンネルの完全なボーン構造

## デプロイとGitHub Pages設定

### 1. リポジトリ作成
```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/YOUR-USERNAME/YOUR-REPOSITORY-NAME.git
git push -u origin main
```

### 2. GitHub Pages有効化
1. リポジトリのSettingsに移動
2. 「Pages」セクションで「Deploy from a branch」を選択
3. ブランチを「main」、フォルダを「/ (root)」に設定
4. 「Save」をクリック

### 3. デモアクセス
`https://YOUR-USERNAME.github.io/YOUR-REPOSITORY-NAME/` でデモにアクセス可能

## 技術的な課題と解決策

### 1. CORS問題
**問題**: ローカルファイルアクセス時のCORS制限
**解決**: ローカルWebサーバーの使用を推奨

```bash
# Python
python -m http.server 8000

# Node.js
npx serve .

# PHP
php -S localhost:8000
```

### 2. アニメーション互換性
**問題**: VRMAファイルとVRMモデルの互換性確保
**解決**: VRM 1.0ヒューマノイド仕様に完全準拠したアニメーション作成

### 3. パフォーマンス最適化
**問題**: 大きなVRMファイルでの描画性能
**解決**: VRMUtilsを使用した最適化処理

```javascript
VRMUtils.removeUnnecessaryVertices(gltf.scene);
VRMUtils.combineSkeletons(gltf.scene);
VRMUtils.combineMorphs(vrm);
```

## ブラウザ対応状況

- ✅ Chrome 80+
- ✅ Firefox 75+
- ✅ Safari 14+
- ✅ Edge 80+

## カスタマイズ方法

### 新しいアニメーションの追加

1. VRMAファイルを`VRMA/`ディレクトリに配置
2. `index.html`の`VRMA_ANIMATION_URLS`配列を更新
3. HTMLに対応するボタンを追加

```javascript
const VRMA_ANIMATION_URLS = [
    '/VRMA/original_01.vrma',
    '/VRMA/original_02.vrma',
    '/VRMA/your_new_animation.vrma', // 新しいアニメーション
];
```

### UI のカスタマイズ

CSSカスタムプロパティでテーマを簡単に変更できます：

```css
.controls button {
    background: linear-gradient(145deg, #6a82fb, #fc5c7d);
    /* お好みのグラデーションに変更 */
}
```

## まとめ

Three.jsとVRMライブラリを使用することで、比較的簡単にVRMビューアーを作成できました。特に以下の点が重要でした：

1. **VRM 1.0仕様への準拠**: アニメーション互換性のため
2. **パフォーマンス最適化**: VRMUtilsの活用
3. **GitHub Pages**: 簡単なデプロイとデモ共有

VRMエコシステムは今後も発展が期待される分野なので、引き続き技術動向を追っていきたいと思います。

## 参考リンク

- [GitHub Repository](https://github.com/TK-256/vrm-viewer)
- [Live Demo](https://TK-256.github.io/vrm-viewer/)
- [VRM Consortium](https://vrm.dev/)
- [Three.js](https://threejs.org/)
- [@pixiv/three-vrm](https://github.com/pixiv/three-vrm)

## タグ
#Three.js #VRM #WebGL #JavaScript #GitHub Pages #3D #Animation