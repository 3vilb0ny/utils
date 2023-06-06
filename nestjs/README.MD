## Compress Images & Store in Disk

1. Create a pipe

   ```typescript
   import { Injectable, PipeTransform } from "@nestjs/common";
   import { writeFile } from "fs";
   import { extname, join } from "path";
   import sharp from "sharp";

   @Injectable()
   export class SharpPipe
     implements
       PipeTransform<Express.Multer.File, Promise<Express.Multer.File>>
   {
     async transform(file: Express.Multer.File): Promise<Express.Multer.File> {
       const uniqueSuffix = `${Date.now()}-${Math.round(Math.random() * 1e9)}`;
       const ext = extname(file.originalname);
       const filename = `${file.originalname.replace(ext, "")}-${uniqueSuffix}`;

       const imageExtensions = [
         "jpg",
         "jpeg",
         "png",
         "gif",
         "webp",
         "tiff",
         "tif",
         "bmp",
       ];

       let filePath = join(process.env["FILES_DIRECTORY"], filename);

       const isImage = imageExtensions.includes(ext.replace(".", ""));

       if (isImage) {
         filePath += ".webp";
         await sharp(file.buffer)
           .resize(800)
           .webp({ effort: 3 })
           .toFile(filePath);
       } else {
         filePath += ext;
         writeFile(filePath, file.buffer, "binary", (err) => {
           if (err) {
             console.log(
               "There was an error writing the file " + filePath + ext
             );
           }
         });
       }

       file.filename = filename + (isImage ? ".webp" : ext);

       return file;
     }
   }
   ```

2. The controller must inject to the pipe

   ```typescript
   @Post()
   @UseInterceptors(FileInterceptor('file'))
   create(
     @UploadedFile(SharpPipe) file: Express.Multer.File,
     @Body() createMediaDto: CreateMediaDto,
   ) {
     const mediaDto = new CreateMediaDto();
     mediaDto.fileName = file.filename;
     mediaDto.businessId = createMediaDto.businessId;
     return this.mediaService.create(mediaDto);
   }
   ```

3. In App module imports

   ```typescript
   // File management
   // The files are stored in memory, then are transformed and saved in disk by Sharp package
   MulterModule.register({
     storage: memoryStorage(),
   }),
   ```
