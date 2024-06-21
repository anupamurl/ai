describe('Upload function', function() {
    let uploadInstance;

    // Mocking dependencies
    const ContentService = {
        GetCache: jasmine.createSpy('GetCache').and.callFake((key) => {
            const mockData = {
                'BULK_UPLOAD.BULK_FLOW_KEY': 'bulkFlowData',
                'BULK_UPLOAD.BULK_TEMPLATE_KEY': 'bulkTemplateData',
                'BULK_UPLOAD.CSV.TXT': 'csvTemplateData'
            };
            return mockData[key];
        })
    };

    const XLSX = {
        read: jasmine.createSpy('read').and.returnValue({
            SheetNames: ['Sheet1'],
            Sheets: { 'Sheet1': {} }
        }),
        utils: {
            sheet_to_json: jasmine.createSpy('sheet_to_json').and.returnValue([{}])
        }
    };

    // Mock fileReader
    class MockFileReader {
        constructor() {
            this.onload = null;
            this.result = 'mockResult';
        }
        readAsArrayBuffer(file) {
            if (this.onload) {
                this.onload();
            }
        }
    }

    beforeEach(function() {
        uploadInstance = new Upload();
        uploadInstance.fileReader = new MockFileReader();
        uploadInstance.ContentService = ContentService;
        uploadInstance.XLSX = XLSX;
    });

    it('should call GetCache with correct key for bulkFlowData', function() {
        uploadInstance.Upload();
        expect(ContentService.GetCache).toHaveBeenCalledWith('BULK_UPLOAD.BULK_FLOW_KEY');
    });

    it('should call GetCache with correct key for templateType', function() {
        uploadInstance.Upload();
        expect(ContentService.GetCache).toHaveBeenCalledWith('BULK_UPLOAD.TEMPLATE_KEY');
    });

    it('should read file and convert to json', function() {
        uploadInstance.file = new Blob();
        uploadInstance.Upload();
        expect(uploadInstance.fileReader.readAsArrayBuffer).toHaveBeenCalledWith(uploadInstance.file);
        expect(XLSX.read).toHaveBeenCalledWith(jasmine.any(String), { type: 'binary' });
        expect(XLSX.utils.sheet_to_json).toHaveBeenCalled();
    });

    it('should filter out blank objects', function() {
        uploadInstance.Upload();
        expect(uploadInstance.ExcelToJson.length).toBeGreaterThan(0);
    });
});
