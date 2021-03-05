# ������websocket

websocket��URL: /api/ws/search

���̃y�[�W�ł́A�����̂��߂�websocket�v���g�R���ɂ��Đ������܂��B"grep foo"�̌��������������A�G���g���[�f�[�^�̌��������{����ۂɃN���C�A���g�ƃT�[�o�Ԃł��Ƃ肳���JSON�f�[�^�̑S�e��́A [Websocket�����̗�](websocket-search-example.md)�y�[�W�Ō��邱�Ƃ��ł��܂��B

## Ping��Pong�ɂ��keepalive

����websocket�́A�N�G���̃`�F�b�N�A�����̑��M�A�������ʂƌ������v�̎�M�Ɏg�p����܂��B ����websocket�ɂ��āA���b�Z�[�W��"�^�C�v"(Type)���������邽�߂ɂ�RoutingWebsocket �V�X�e�����g�p���邱�Ƃ��z�肳��Ă��܂��B`/api/ws/search` �́A�N�����Ɉȉ��̃��b�Z�[�W�̂悤��"�T�u�^�C�v"(SubType)�܂���"�^�C�v"(Type)�̓o�^����Ă��邱�Ƃ�z�肵�Ă��܂��B�FPONG, parse, search, attach

���F���b�Z�[�W��"�^�C�v"(Type)�́A�����Ȗ��O�Ƃ���"SubProto"�ƌĂ΂�邱�Ƃ�����܂��B����͍���ύX���ꂽ�����ǂ��̂ł����A����API���g�p���ĊJ������ꍇ�A"SubProto"�́A���b�Z�[�W�ł����܂ő��M�����"�^�C�v"(Type)�̒l���w���Ă���ARFC Websocket�T�u�v���g�R���d�l�̂��Ƃł͂Ȃ����Ƃɒ��ӂ��Ă��������B

PONG�^�C�v�̓L�[�v�A���C�u�V�X�e���ł���A�N���C�A���g�͒���I��PINGPONG���N�G�X�g�𑗐M����K�v������܂��B

����́A���[�U�������v�����v�g�������Ɍ������Ď��g��ł���Ƃ��ɁA���̗����Őڑ�������ɕۂ���Ă邩�ǂ��������[�U�ɓ`���邽�߂Ɏg�����Ƃ��ł��܂��BWebsocket ���̂������Ă��邩�ǂ����𒲂ׂ邾���Ȃ̂ŁA���[�U����������鎞�ɂ͑S���K�v�Ȃ���������܂���B

## ������"parse"

"parse"�̃^�C�v�́A�����o�b�N�G���h�����ۂɌĂяo�����ƂȂ��A�N�G�����K�����ǂ������葁���m�F���邽�߂Ɏg�p����܂��B

�K���ȃN�G���̃��N�G�X�g�ƃ��X�|���X�̏ꍇ�ł́A���̂悤�� JSON �������܂�:

�t�����g�G���h����̃��N�G�X�g:
```json
{
        SearchString: "tag=apache grep firefox | regex "Firefox(<version>[0-9]+) .+" | count by version""
}
```

�o�b�N�G���h����̃��X�|���X:
```
{
        GoodQuery: true,
        ParseQuery: "tag=apache grep firefox | regex "Firefox(<version>[0-9]+) .+" | count by version"",
        ModuleIndex: 0,
}
```

�s�K���ȃN�G���̃��N�G�X�g�ƃ��X�|���X�̏ꍇ�ł́A���̂悤�� JSON �������܂�:

�t�����g�G���h����̃��N�G�X�g:
```
{
        SearchString: "tag=apache grep firefox | MakeRainbows",
}
```

�o�b�N�G���h����̃��X�|���X:
```
{
        GoodQuery: false,
        ParseError: "ModuleError: MakeRainbows is not a valid module",
        ModuleIndex: 1,
}
```

## �����̏�����
���ׂĂ̌����̓E�F�u�\�P�b�g����ď���������A���̊J�n���� "parse"�A"PONG"�A"search"�A"attach "�̃T�u�^�C�v���w�肷�邱�Ƃ��K�{�ł��B 

���̎葱���́Awebsocket �̊m�����Ɉȉ��� JSON �𑗐M���邱�Ƃōs���܂�:
```
{"Subs":["PONG","parse","search","attach"]}
```

SearchString �����o�ɂ́A�������Ăяo�����ۂ̃N�G�����܂܂�Ă��Ȃ���΂Ȃ�܂���B

SearchStart��SearchEnd�ɂ́A�N�G�������삷�鎞�Ԕ͈͂��w�肷��K�v������܂��B �͈͎w��̎����́A"2006-01-02T15:04:05.9999999Z07:00 "�̂悤��RFC3339Nano�`���̏����ł��B

�K���ȃN�G�������������N�G�X�g�̗�Ƃ��ẮA���̂悤�� JSON ���������܂�:
```
{
       SearchString: "tag=apache grep firefox | nosort",
       SearchStart:  "2015-01-01T12:01:00.0Z07:00",
       SearchEnd:    "2015-01-01T12:01:30.0Z07:00",
       Background:   false,
}
```

//�������N�[���ł���΁A�T�[�o��Yay/Nay�ƐV�����T�u�^�C�v�ɉ������܂��B
//searchStart �� searchEnd ��RFC3339Nano �`���̕�����Ŏw�肳��܂�

�K�؂ȃN�G���ɑ΂��郌�X�|���X�͗Ⴆ�Ύ��̂悤��JSON�ɂȂ�܂�:
```
{
        SearchString: "tag=apache grep firefox | nosort",
        RenderModule: "text",
        RenderCmd:    "text",
        OutputSearchSubproto:  "searchSDF8973",
        OutputStatsSubproto:   "statsSDF8973",
        SearchID:              "skdlfjs9098",
		SearchStartRange:      "2015-01-01T12:01:00.0Z07:00",
        SearchEndRange:        "2015-01-01T12:01:30.0Z07:00",
        Background:            false,
}
```

�G���[�̃��X�|���X��JSON�͎��̂悤�ɂȂ�܂�:
```
{
        Error: "Search error: The parameter "ChuckTesta" is invalid",
}
```

�K���Ȍ������N�G�X�g�ւ̃��X�|���X�ł́A�N���C�A���g�͌���ACK��t���ă��X�|���X���܂��BAck ���X�|���X�� true �� false �̂ǂ��炩�ł��B false ���X�|���X�́A�t�����g�G���h�������ł��Ȃ������_�����O���W���[�����o�b�N�G���h�����N�G�X�g�����ꍇ�Ɏg�p����܂��B����́A�Ⴆ�΁A�t�����g�G���h�ƃo�b�N�G���h�̊ԂŃo�[�W�����s��v������ꍇ�ɋN����܂��B

���� JSON �́A�O�̉�����Ƒ΂ɂȂ�Atrue ��ACK �̗��\���܂�:
```
{
       Ok: True,
       OutputSearchSubproto: "searchSDF8973"
}
```

ACK �����M�����ƁA�o�b�N�G���h�͌������J�n���A�V�����T�u�^�C�v�̌������ʂ̒񋟂��J�n���܂��B ����"search"�A"parse", "PONG"�̃T�u�^�C�v�̓A�N�e�B�u�Ȃ܂܂ŁA�t�����g�G���h�͐V�����N�G�����`�F�b�N������A�ǉ��̌������J�n�����肷�邽�߂Ɏg�p���邱�Ƃ��ł��܂��B �������A�A�N�e�B�u�ȃN�G���Ƃ̂��Ƃ�͂��ׂāA�V�����l�S�V�G�[�g���ꂽ�����ŗL�̃T�u�^�C�v����čs����K�v������܂��B

## ����
���ׂĂ̌����͊��S�ɔ񓯊��ł����A�������o�b�N�O���E���h��Ԃɂ��邱�Ƃ����N�G�X�g���Ȃ��܂܃N���C�A���g���ؒf���ꂽ��A�ڑ����N���b�V�������肵���ꍇ�A�A�N�e�B�u�Ȍ����͏I�����A�f�[�^�̓S�~���s���ɂȂ�܂��B ����́A���\�[�X�̌͊���h�����߂ł��B ���[�U�[�̓o�b�N�O���E���h�������邱�Ƃ𖾎��I�ɗv�����Ȃ���΂Ȃ�܂���B

�����̗��p�҂���̌��������p���邱�Ƃ��ł��܂��B �Ⴆ�΁A�{�u���n�߂������ɂ��āA�W���l�b�g���A�^�b�`���Č��ʂ����邱�Ƃ��ł��܂��B �o�b�N�O���E���h�ł͂Ȃ������́A���ׂĂ̗��p�҂���ؒf���ꂽ�ꍇ�ɂ̂ݏI�����A�N���[���A�b�v����܂��B �܂�A�{�u���������J�n���ăW���l�b�g���A�^�b�`���Ă��鎞�A�{�u���u���E�U�������ړ������肵���ꍇ���A�����͏I�����܂���B �W���l�b�g�͂��̌����ł̍�Ƃ𑱂��邱�Ƃ��ł��܂��B �������A��������ɃW���l�b�g���ړ�������u���E�U������肵���ꍇ�́A�����͏I�����A�S�~��������邱�ƂɂȂ�܂��B

## �A�N�e�B�u�Ȍ������ɏo�͂���铝�v���

���v���́A���v���� ID ���g���ă��N�G�X�g����܂��B

## ���N�G�X�g/���X�|���XID �̎Q��

���N�G�X�g�ƃ��X�|���X��ID�R�[�h�̈ꗗ:
```
{
    req: {
        REQ_CLOSE: 0x1,
        REQ_ENTRY_COUNT: 0x3,
        REQ_DETAILS: 0x4,
        REQ_TAGS: 0x5,
        REQ_STATS_SIZE: 0x7F000001, //gets backend "size" value of stats chunks. never used
        REQ_STATS_RANGE: 0x7F000002, //gets current time range covered by stats. rarely used
        REQ_STATS_GET: 0x7F000003, //gets stats sets over all time. may be used initially
        REQ_STATS_GET_RANGE: 0x7F000004, //gets stats in a specific range
        REQ_STATS_GET_SUMMARY: 0x7F000005, //gets stats summary for entire results
        REQ_STATS_GET_LOCATION: 0x7F000006, //get current timestamp for search progress
        REQ_GET_ENTRIES: 0x10, //1048578
        REQ_STREAMING: 0x11,
        REQ_TS_RANGE: 0x12,
		REQ_GET_EXPLORE_ENTRIES: 0xf010,
		REQ_EXPLORE_TS_RANGE: 0xf012,
        SEARCH_CTRL_CMD_DELETE: 'delete',
        SEARCH_CTRL_CMD_ARCHIVE: 'archive',
        SEARCH_CTRL_CMD_BACKGROUND: 'background',
        SEARCH_CTRL_CMD_STATUS: 'status'
    },
    rep: {
        RESP_CLOSE: 0x1,
        RESP_ENTRY_COUNT: 0x3,
        RESP_DETAILS: 0x4,
        RESP_TAGS: 0x5,
        RESP_STATS_SIZE: 0x7F000001, //2130706433
        RESP_STATS_RANGE: 0x7F000002, //2130706434
        RESP_STATS_GET: 0x7F000003, //2130706435
        RESP_STATS_GET_RANGE: 0x7F000004, //2130706436
        RESP_STATS_GET_SUMMARY: 0x7F000005,
        RESP_STATS_GET_LOCATION: 0x7F000006, //2130706438
        RESP_GET_ENTRIES: 0x10,
        RESP_STREAMING: 0x11,
        RESP_TS_RANGE: 0x12,
		RESP_GET_EXPLORE_ENTRIES: 0xf010,
		RESP_EXPLORE_TS_RANGE: 0xf012,
        RESP_ERROR: 0xFFFFFFFF,
        SEARCH_CTRL_CMD_DELETE: 'delete',
        SEARCH_CTRL_CMD_ARCHIVE: 'archive',
        SEARCH_CTRL_CMD_BACKGROUND: 'background',
        SEARCH_CTRL_CMD_STATUS: 'status'
    }
}
```
