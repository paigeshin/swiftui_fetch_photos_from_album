# swiftui_fetch_photos_from_album

```swift
//
//  ContentView.swift
//  LoadPhotos
//
//  Created by paige shin on 2023/04/21.
//

import SwiftUI
import Photos

enum SelectedPhotos: Int {
    case all = 1
    case favorites = 0
}

struct Photo: Identifiable {
    let id = UUID()
    var photo: Image
}

struct ContentView: View {
    
    @State private var photos = [Photo]()
    @State private var selectedPhotos = SelectedPhotos.favorites
    
    var body: some View {
        VStack {
            
            Picker("Select Photos", selection: self.$selectedPhotos) {
                Text("Favorites").tag(SelectedPhotos.favorites)
                Text("All").tag(SelectedPhotos.all)
            }
            .pickerStyle(SegmentedPickerStyle())
            .onChange(of: self.selectedPhotos) { selectedPhotos in
                switch selectedPhotos {
                case .all:
                    self.photos.removeAll()
                    self.fetchPhotos()
                case .favorites:
                    self.photos.removeAll()
                    self.fetchFavorites()
                }
            }
            
            ScrollView(showsIndicators: false) {
                let grid = GridItem(.flexible(minimum: 0), spacing: 0)
                LazyVGrid(columns: [grid, grid, grid], spacing: 0) {
                    ForEach(self.photos) { photo in
                        photo
                            .photo
                            .resizable()
                            .aspectRatio(1, contentMode: .fill)
                            .border(Color.white)
                    }
                }
            }
            
//            List {
//                ForEach(self.photos) { photo in
//                    photo
//                        .photo
//                        .resizable()
//                        .aspectRatio(1, contentMode: .fill)
//                        .border(Color.white)
//                }
//            }
        }
        .onAppear(perform: self.askPermission)
    }
    
    private func fetchPhotos() {
        let imgManager = PHImageManager.default()
        let requestOptions = PHImageRequestOptions()
        requestOptions.isSynchronous = true
        requestOptions.deliveryMode = .highQualityFormat
        let fetchOptions = PHFetchOptions()
        fetchOptions.sortDescriptors = [NSSortDescriptor(key: "creationDate", ascending: true)]
        let fetchResult: PHFetchResult = PHAsset.fetchAssets(with: .image, options: fetchOptions)
        if fetchResult.count > 0 {
            for i in 0 ..< fetchResult.count {
                imgManager.requestImage(for: fetchResult.object(at: i), targetSize: CGSize(width: 200, height: 200), contentMode: .aspectFit, options: requestOptions) { uiImage, _ in
                    if let uiImage {
                        let photo = Photo(photo: Image(uiImage: uiImage))
                        self.photos.append(photo)
                    }
                }
            }
        } else {
            // Photo is empty
        }
    }
    
    private func fetchFavorites() {
        let fetchOptions = PHFetchOptions()
        let imgManager = PHImageManager.default()
        let requestOptions = PHImageRequestOptions()
        requestOptions.isSynchronous = true
        requestOptions.deliveryMode = .highQualityFormat
        fetchOptions.predicate = NSPredicate(format: "title = %@", "Favorites")
        fetchOptions.sortDescriptors = [NSSortDescriptor(key: "creationDate", ascending: true)]
        let favorites: PHFetchResult = PHAssetCollection.fetchAssetCollections(with: .smartAlbum, subtype: .smartAlbumFavorites, options: nil)
        var assetCollection = PHAssetCollection()
        if let firstObject = favorites.firstObject {
            assetCollection = firstObject
        }
        let photoAssets = PHAsset.fetchAssets(in: assetCollection, options: nil)
        
        photoAssets
            .enumerateObjects { asset, numbers, pointer in
                imgManager.requestImage(for: asset, targetSize: CGSize(width: 100, height: 200), contentMode: .aspectFit, options: requestOptions) { uiImage, _ in
                    if let uiImage {
                        let photo = Photo(photo: Image(uiImage: uiImage))
                        self.photos.append(photo)
                    }
                }
            }
    }
    
    private func askPermission() {
        PHPhotoLibrary.requestAuthorization(for: .readWrite) { status in
            if status == .authorized {
                switch self.selectedPhotos {
                case .all:
                    self.fetchPhotos()
                case .favorites:
                    self.fetchFavorites()
                }
            }
        }
    }
    
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}

```
