```swift
private func getBitrate(with asset: AVURLAsset) -> Int {
        guard let originBitrate = asset.tracks(withMediaType: .video).first?.estimatedDataRate else {
            return self.maxBitrate
        }
        
        let originBitrateIntValue = Int(originBitrate)
        
        guard originBitrateIntValue < self.maxBitrate else { return self.maxBitrate }
        return originBitrateIntValue
    }
    
    private func getFps(with asset: AVURLAsset) -> Float {
        guard let originFps = asset.tracks(withMediaType: .video).first?.nominalFrameRate else {
            return self.maxFps
        }
        
        return originFps <= self.maxFps ? originFps : self.maxFps
    }
    
    private func getScale(with asset: AVURLAsset) -> CGSize {
        guard let originSize = asset.tracks(withMediaType: .video).first?.naturalSize else {
            return self.maxResoultion
        }
        
        // 영상 원본 비율
        let ratio = originSize.width / originSize.height
        
        // 비율에 따라 조정할 차원 결정
        let isWidthDominant = ratio > 1
        
        // 조정 비율과 한계치 결정
        let limit = isWidthDominant ? self.maxWidth : self.maxHeight
        let currentDimension = isWidthDominant ? originSize.width : originSize.height
        
        // 해상도 조정 필요 여부 확인
        guard currentDimension > limit else { return originSize }
        
        let decreaseRatio = limit / currentDimension
        return isWidthDominant ? CGSize(width: limit, height: originSize.height * decreaseRatio) : CGSize(width: originSize.width * decreaseRatio, height: limit)
    }
```