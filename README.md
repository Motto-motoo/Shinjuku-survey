import { useState, useEffect } from "react";
import { getStorage, ref, listAll, getDownloadURL } from "firebase/storage";
import { initializeApp } from "firebase/app";

// Firebase 設定（環境変数を使用）
const firebaseConfig = JSON.parse(process.env.REACT_APP_FIREBASE_CONFIG);
const app = initializeApp(firebaseConfig);
const storage = getStorage(app);

export default function RandomImage() {
  const [imageURL, setImageURL] = useState(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    const fetchRandomImage = async () => {
      setLoading(true);
      const storageRef = ref(storage, "images"); // "images" は Firebase Storage のフォルダ名
      const result = await listAll(storageRef);
      
      if (result.items.length === 0) {
        setLoading(false);
        return;
      }
      
      // ランダムに1枚選択
      const randomIndex = Math.floor(Math.random() * result.items.length);
      const imageRef = result.items[randomIndex];
      const url = await getDownloadURL(imageRef);
      
      setImageURL(url);
      setLoading(false);

      // 30秒後に画像を非表示
      setTimeout(() => setImageURL(null), 30000);
    };
    
    fetchRandomImage();
  }, []);

  return (
    <div>
      {loading ? <p>Loading...</p> : imageURL ? <img src={imageURL} alt="Random" style={{ maxWidth: "100%" }} /> : <p>Image hidden</p>}
    </div>
  );
}
