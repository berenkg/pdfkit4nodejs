### Bu belge node.js uygulamalarında pdf çıktısı olarak raporlama yapmak isteyen geliştiricilere türkçe kaynak olması için hazırlanmıştır.

# Başlarken
İlk olarak aşağıdaki kodu alın ve çalıştırın. Bunun için `npm install pdfkit --save` modülünü projenize dahil etmeniz gerekmektedir çünkü anlatacağımız modül `pdfkit` modülüdür.

```js
const PDFDocument = require('pdfkit');
const fs = require('fs');
 
// Create a document
const doc = new PDFDocument();
 
// Pipe its output somewhere, like to a file or HTTP response
// See below for browser usage
doc.pipe(fs.createWriteStream('output.pdf'));
 
// Embed a font, set the font size, and render some text
doc.font('fonts/paltno.ttf')// font
   .fontSize(25)// boyut
   .text('Some text with an embedded font!', 25, 25);// yukarıdan ve soldan 25 uzaklıkta.

doc.end();
```
Yukarıdaki kod çalıştırıldığında çalışmakta olduğunuz dizin altına output.pdf şeklinde bir pdf dosyası oluşturulacaktır ve içerisinde sadece 1 adet metin olacaktır. Font kısmını projenize dahil etmeniz gerekiyor. Gerekli dosyaları repoda paylaştım.

# Yeni Bir Sayfa Eklemek
Projeye başladığınızda otomatik olarak yeni sayfanız eklenmektedir. Diğer sayfaları eklemek için bir metotu çağırmanız gerekiyor:  `doc.addPage();` 
Sayfa eklemesi yaparken sayfanın boyutlarını ayarlamanızda mümkün. Aşağıdaki koda bakınız.
```js
doc.addPage({
    margins: { top: 50,
    bottom: 50,
    left: 72,
    right: 72
    }
});
```
Peki sürekli her sayfa eklemesinde bu işleri gerçekleştirecek miyiz? Hayır bir fonksiyon yardımıyla bunun önüne geçebiliriz.
```js
const PDFDocument = require('pdfkit');
const fs = require('fs');
 
// Create a document
const doc = new PDFDocument();
 
// Pipe its output somewhere, like to a file or HTTP response
// See below for browser usage
doc.pipe(fs.createWriteStream('ustbaslik.pdf'));
 
// Embed a font, set the font size, and render some text
doc.font('fonts/paltno.ttf')
   .fontSize(25)
   .text('Some text with an embedded font!', 25, 25);


// Bu fonksyion sayesinde yeni sayfa oluşturulduğunda yapılmasını istediğimiz özelikleri direkt olarak ekleyebiliriz.
yeniSayfa();

// Finalize PDF file
doc.end();

function yeniSayfa(){
    doc.addPage({
        margins: { top: 50,
        bottom: 50,
        left: 72,
        right: 72
        }
        });
    doc.font('fonts/paltno.ttf')
    .fontSize(25)
    .text('Yeni Sayfa');

}
```


### Not:
```sh
events.js:174
      throw er; // Unhandled 'error' event
      ^

Error: UNKNOWN: unknown error, open 'C:\Users\bkgorun\Desktop\pdfkit_example\ustbaslik.pdf'
Emitted 'error' event at:
    at errorOrDestroy (internal/streams/destroy.js:107:12)
    at WriteStream.onerror (_stream_readable.js:717:7)
    at WriteStream.emit (events.js:198:13)
    at lazyFs.open (internal/fs/streams.js:277:12)
    at FSReqWrap.args [as oncomplete] (fs.js:140:20)
```
Çalışmalarınız esnasında yukarıdakine benzer bir hata alırsanız. Oluşturulacak dosya ismini değiştirmenizi öneririm. Ya da var olan dosyasyı bir pdf görüntüleyicisi ile görüntülediğinizden dolayı üzerinde değişiklik yapmanıza izin verilmemektedir.

# Sayfalar Arasında Gezinmek
Bu işlemi gerçekleştirmek için ilk olarak doc nesnesi yaratılırken bir parametre berlitmeniz gerekmektedir. Aşağıdaki gibi
```js
const doc = new PDFDocument(
    { 
        bufferPages: true
    }
);
```
artık sayfalar arasında gezinmek *pek tabii* mümkündür. Aşağıdaki örneği inceleyerek durumu daha iyi anlayabiliriz. Sayfalar oluşturduktan sonra sayfa numarası ekleme işlemi yapılmak istenilebilir. Bunun için kabaca aşağıdaki yapı kullanılabilir.
```js
const PDFDocument = require('pdfkit');
const fs = require('fs');
const doc = new PDFDocument(
    { 
        bufferPages: true // Sayfalar arasında gezinebilmek için
    }
);
doc.pipe(fs.createWriteStream('ustbaslik.pdf'));
doc.font('fonts/paltno.ttf')
   .fontSize(25)
   .text('Some text with an embedded font!', 25, 25);

yeniSayfa();
yeniSayfa();
yeniSayfa();
yeniSayfa();
yeniSayfa();
const range = doc.bufferedPageRange(); // sayfa sayısı
let baslangic =  range.start; // 0
let son = range.count + baslangic; // sayfa sayısı
for (let i = baslangic; i < son; i++) {
    doc.switchToPage(i); // ilgili sayfaya geçiş
    doc.text(`Page ${i + 1} of ${range.count}`);
    }
doc.end();
function yeniSayfa(){
    doc.addPage({
        margins: { top: 50,
        bottom: 50,
        left: 72,
        right: 72
        }
        });
    doc.font('fonts/paltno.ttf')
    .fontSize(25)
    .text('Yeni Sayfa');

}
```

# Belge Meta Verilerini Ayarlama
PDF belgelerinde, belgenin başlığı veya yazarı gibi bunlarla ilişkili çeşitli meta veriler bulunabilir. Bu bilgileri doc.info nesnesine ekleyerek veya oluşturma sırasında bir info nesnesini belgeye geçirerek ekleyebilirsiniz.
Belge meta verilerine ekleyebileceğiniz tüm özelliklerin listesi aşağıdadır. PDF spesifikasyonuna göre, her mülkün ilk harfini büyük harfle yazması gerekir.
1. Title 
2. Author
3. Subject 
4. Keywords
5. CreationDate
6. ModDate
Örnek kullanım aşağıda gösterilmiştir.
```js
const doc = new PDFDocument(
    { 
        bufferPages: true, // Sayfalar arasında gezinebilmek için
        info: {
            Title: "Üst Başlık",
            Author: "Beren Kuday GÖRÜN",
            Subject : "yok YOK yok"
        }
    }
);
```
# Vektörleri Kullanmak
Daha öncesinde html çalışmalarınıda canvas modelini ya da svg yapısını kullandıysanız bu kısmı çok rahat anlayacaksınızdır.
Basit bir örnekle başlayalım...
```js
doc.moveTo(0, 20)	// set the current point
    .lineTo(100, 160)	// draw a line
    .quadraticCurveTo(130, 200, 150, 120)	// draw a quadratic curve
    .bezierCurveTo(190, -40, 200, 200, 300, 150) // draw a bezier curve
    .lineTo(400, 90)	// draw another line
    .stroke();	// stroke the path

```
Yukarıdaki kodlar eğrilerden oluşan bir doğru parçası oluşturacaktır. Bunun yanında diğer geometrik şekilleride çizebiliriz.
Yukarıdaki şekli tabikide svg yapısı ilede çizmek mümkündür. Aşağıdaki kod ile yukarıdaki aynı işlemi görmektedir.
```js
doc.path('M 0,20 L 100,160 Q 130,200 150,120 C 190,-40 200,200 300,150 L 400,90')
.stroke()
```
## Diğer yapılar
1. rect(x, y, width, height)
2. roundedRect(x, y, width, height, cornerRadius) 
3. ellipse(centerX, centerY, radiusX, radiusY = radiusX) 
4. circle(centerX, centerY, radius)
5. polygon(points...)
Örnek bir kod:
```js
doc.polygon([100, 0], [50, 100], [150, 100]);
doc.stroke();

```

# Yazı Yazma
Aşağıdaki koda bir bakın ve tahmin edin. Bence ne yaptığını çok rahat anlayacaksınız.
```js
const lorem = 'Lorem ipsum dolor sit amet, consectetur adipiscing elit. Etiam in suscipit purus.	Vestibulum ante ipsum primis in faucibus orci luctus et ultrices posuere cubilia Curae; Vivamus nec hendrerit felis. Morbi aliquam facilisis risus eu lacinia. Sed eu leo in turpis fringilla hendrerit. Ut nec accumsan nisl.';

doc.fontSize(8);
doc.text(`This text is left aligned. ${lorem}`, { width: 410,
align: 'left'
}
);

doc.moveDown();
doc.text(`This text is centered. ${lorem}`, { width: 410,
align: 'center'
}
);

doc.moveDown();
doc.text(`This text is right aligned. ${lorem}`, { width: 410,
align: 'right'
}
);

doc.moveDown();
doc.text(`This text is justified. ${lorem}`, { width: 410,
align: 'justify'
}
);
```
Başka bir örnek:
```js
const lorem = `Lorem ipsum dolor sit amet, consectetur adipiscing elit. Etiam in suscipit purus. Vestibulum ante ipsum primis in faucibus orci luctus et ultrices posuere cubilia Curae; Vivamus nec hendrerit felis. Morbi aliquam facilisis risus eu lacinia. Sed eu leo in turpis fringilla hendrerit. Ut nec accumsan nisl. Suspendisse rhoncus nisl posuere tortor tempus et dapibus elit porta. Cras leo neque, elementum a rhoncus ut, vestibulum non nibh. Phasellus pretium justo turpis. Etiam vulputate, odio vitae tincidunt ultricies, eros odio dapibus nisi, ut tincidunt lacus arcu eu elit. Aenean velit erat, vehicula eget lacinia ut, dignissim non tellus. Aliquam nec lacus mi, sed vestibulum nunc. Suspendisse potenti. Curabitur vitae sem turpis.
Vestibulum sed neque eget dolor dapibus porttitor at sit amet sem. Fusce a turpis lorem. Vestibulum ante ipsum primis in faucibus orci luctus et ultrices posuere cubilia Curae;`;

doc.text(lorem, { columns: 3,
columnGap: 15,
height: 100,
width: 465, align: 'justify'
});

```
Başka bir örnek: 
```js
doc.fillColor('green')
    .text(lorem.slice(0, 500), {
    width: 465, continued: true
    }).fillColor('red')
    .text(lorem.slice(500));
```

# Fotoğraflar
```js
// Scale proprotionally to the specified width doc.image('images/test.jpeg', 0, 15, {width: 300})
.text('Proportional to width', 0, 0);

// Fit the image within the dimensions doc.image('images/test.jpeg', 320, 15, {fit: [100, 100]})
.rect(320, 15, 100, 100)
.stroke()
.text('Fit', 320, 0);

// Stretch the image
doc.image('images/test.jpeg', 320, 145, {width: 200, height: 100})
.text('Stretch', 320, 130);

// Scale the image
doc.image('images/test.jpeg', 320, 280, {scale: 0.25})
.text('Scale', 320, 265);

// Fit the image in the dimensions, and center it both horizontally and vertically doc.image('images/test.jpeg', 430, 15, {fit: [100, 100], align: 'center', valign: 'center'})
.rect(430, 15, 100, 100).stroke()
.text('Centered', 430, 0);

```
## Not
İnternet üzerindeki çalışmalarınızda eğer grafikleri de pdf'inize eklemek isterseniz bunun için base64 ile şifrelenmiş bir şekilde `doc.image()` yapısını kullanabilirsiniz. Basit bir örnek aşağıda verilmiştir.
```js
var dataa = `data:image/png;base64,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`;
doc.image(dataa, {width: 400});
```
Temel açıdan pdfkit'i kullanmayı gördük. İnternetteki örnekleri artık daha iyi bir şekilde anlayıp projelerinizde kullanabilirsiniz.


