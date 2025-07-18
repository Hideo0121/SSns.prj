# ワイヤーフレーム

このセクションでは、アプリケーションの各画面のワイヤーフレームを記述します。レイアウト、コンポーネント配置、機能要素の位置関係を明確にします。

## ワイヤーフレーム作成ガイドライン

- **画面サイズ**: デスクトップ (1920x1080)、タブレット (768x1024)、スマートフォン (375x667) を想定
- **グリッドシステム**: 12カラムグリッドを基準とした配置
- **UI要素**: Material-UIコンポーネントを前提とした設計
- **レスポンシブ**: 各デバイスサイズでの表示を考慮

---

## 1. 認証画面

### レイアウト概要
```html
<!-- Material-UI LoginPage Mockup -->
<div style="min-height: 100vh; background: linear-gradient(135deg, #1565c0 0%, #1976d2 100%);">
  
  <!-- Header -->
  <AppBar position="static" elevation={0} style="background: transparent;">
    <Toolbar>
      <Box display="flex" alignItems="center">
        <Avatar src="/logo.svg" sx={{ mr: 2, width: 40, height: 40 }} />
        <Typography variant="h6" color="white" fontWeight="bold">
          スタッフ管理システム
        </Typography>
      </Box>
    </Toolbar>
  </AppBar>

  <!-- Main Content -->
  <Container maxWidth="sm" style="padding-top: 10vh;">
    <Card elevation={8} style="border-radius: 16px; padding: 48px;">
      
      <!-- Login Form Header -->
      <Box textAlign="center" mb={4}>
        <Avatar sx={{ mx: 'auto', mb: 2, bgcolor: '#1976d2', width: 56, height: 56 }}>
          <LockOutlinedIcon />
        </Avatar>
        <Typography variant="h4" fontWeight="bold" color="#1565c0">
          ログイン
        </Typography>
        <Typography variant="body2" color="text.secondary" mt={1}>
          システムにアクセスするには認証が必要です
        </Typography>
      </Box>

      <!-- Login Form -->
      <form>
        <TextField
          fullWidth
          variant="outlined"
          label="ユーザーコード"
          placeholder="例：user001"
          margin="normal"
          InputProps={{
            startAdornment: <PersonIcon color="action" style="margin-right: 8px;" />
          }}
          error={hasError}
          helperText={errorMessage}
        />
        
        <TextField
          fullWidth
          type="password"
          variant="outlined"
          label="パスワード"
          placeholder="英数字を含む8文字以上"
          margin="normal"
          InputProps={{
            startAdornment: <LockIcon color="action" style="margin-right: 8px;" />
          }}
        />

        <Button
          fullWidth
          variant="contained"
          size="large"
          style="margin-top: 24px; padding: 12px; border-radius: 8px; background: linear-gradient(45deg, #1565c0 30%, #1976d2 90%);"
          startIcon={<LoginIcon />}
        >
          ログイン
        </Button>
      </form>

      <!-- Error Alert -->
      <Collapse in={showError}>
        <Alert severity="error" style="margin-top: 16px;" icon={<ErrorIcon />}>
          ユーザーコードまたはパスワードが正しくありません
        </Alert>
      </Collapse>

    </Card>
  </Container>

  <!-- Footer -->
  <Box position="fixed" bottom={0} width="100%" p={2}>
    <Typography variant="caption" color="white" textAlign="center" display="block">
      © 2025 スタッフ管理システム. All rights reserved.
    </Typography>
  </Box>

</div>
```

### コンポーネント詳細

**Header**
- アプリケーション名/ロゴ
- 高さ: 64px
- 背景色: 青の濃淡 (#1565c0 - #1976d2)
- テキスト色: 白 (#ffffff)
- グラデーション効果: 左から右へ濃い青→薄い青

**ログインフォーム**
- 中央配置 (max-width: 400px)
- Material-UI Card コンポーネント
- Shadow elevation: 3

**入力フィールド**
- TextField (Material-UI)
- バリデーション表示（日本語メッセージ）
- user_code: 半角英数字、必須、重複チェック
  - placeholder: "例：user001"
- password: 8文字以上、英数字含む、必須
  - placeholder: "英数字を含む8文字以上"
- エラー表示: フィールド直下に赤文字で日本語メッセージ

**ログインボタン**
- Button (Material-UI)
- variant: contained
- color: primary
- fullWidth: true

**認証・セッション管理**
- **認証方式: Laravel Sanctum（Personal Access Token）**
- トークン有効期限: 3時間
- トークン自動更新: アクティブ時に残り30分で自動延長
- セッション切れ時: 自動的に認証画面へリダイレクト
- セッション警告: 有効期限10分前にアラート表示

---

## 2. スタッフ検索・スタッフ一覧画面

### レイアウト概要
```html
<!-- Material-UI Staff List Page Mockup -->
<div style="min-height: 100vh; background: #f3f7ff;">
  
  <!-- Header -->
  <AppBar position="static" style="background: linear-gradient(90deg, #1565c0 0%, #1976d2 100%);">
    <Toolbar>
      <Box display="flex" alignItems="center" flexGrow={1}>
        <Avatar src="/logo.svg" sx={{ mr: 2, width: 32, height: 32 }} />
        <Typography variant="h6" color="white" fontWeight="bold">
          スタッフ管理システム
        </Typography>
      </Box>
      <Box display="flex" alignItems="center" gap={2}>
        <Chip 
          avatar={<Avatar>山</Avatar>}
          label="山田太郎 (管理者)"
          variant="outlined"
          style="color: white; borderColor: white;"
        />
        <Button color="inherit" startIcon={<LogoutIcon />}>
          ログアウト
        </Button>
      </Box>
    </Toolbar>
  </AppBar>

  <!-- Main Content -->
  <Container maxWidth="lg" style="padding: 24px;">
    
    <!-- Search and Filter Area -->
    <Paper elevation={2} style="padding: 24px; margin-bottom: 24px; border-radius: 12px;">
      <Typography variant="h6" gutterBottom color="#1565c0" fontWeight="bold">
        スタッフ検索・フィルター
      </Typography>
      
      <Grid container spacing={3} alignItems="center">
        <Grid item xs={12} md={4}>
          <TextField
            fullWidth
            variant="outlined"
            placeholder="氏名、メールアドレス、電話番号で検索"
            InputProps={{
              startAdornment: <SearchIcon color="action" style="margin-right: 8px;" />
            }}
          />
        </Grid>
        <Grid item xs={12} md={2}>
          <FormControl fullWidth>
            <InputLabel>役職</InputLabel>
            <Select label="役職">
              <MenuItem value="all">すべて</MenuItem>
              <MenuItem value="admin">管理者</MenuItem>
              <MenuItem value="staff">スタッフ</MenuItem>
            </Select>
          </FormControl>
        </Grid>
        <Grid item xs={12} md={2}>
          <FormControl fullWidth>
            <InputLabel>ソート</InputLabel>
            <Select label="ソート">
              <MenuItem value="name">氏名順</MenuItem>
              <MenuItem value="created">登録日順</MenuItem>
              <MenuItem value="updated">更新日順</MenuItem>
            </Select>
          </FormControl>
        </Grid>
        <Grid item xs={12} md={4}>
          <Box display="flex" gap={1}>
            <Button variant="contained" startIcon={<AddIcon />} style="background: #1976d2;">
              新規登録
            </Button>
            <Button variant="outlined" startIcon={<EmailIcon />} disabled={selectedCount === 0}>
              メール送信 ({selectedCount})
            </Button>
          </Box>
        </Grid>
      </Grid>
    </Paper>

    <!-- Staff List Table -->
    <Paper elevation={2} style="border-radius: 12px; overflow: hidden;">
      <Box p={2} borderBottom="1px solid #e0e0e0">
        <Typography variant="h6" color="#1565c0" fontWeight="bold">
          スタッフ一覧 (150名)
        </Typography>
      </Box>
      
      <TableContainer>
        <Table>
          <TableHead style="background: #f5f5f5;">
            <TableRow>
              <TableCell padding="checkbox">
                <Checkbox />
              </TableCell>
              <TableCell><b>ID</b></TableCell>
              <TableCell><b>氏名</b></TableCell>
              <TableCell><b>役職</b></TableCell>
              <TableCell><b>メール</b></TableCell>
              <TableCell><b>電話番号</b></TableCell>
              <TableCell><b>登録日</b></TableCell>
              <TableCell><b>操作</b></TableCell>
            </TableRow>
          </TableHead>
          <TableBody>
            <TableRow hover style="cursor: pointer;">
              <TableCell padding="checkbox">
                <Checkbox />
              </TableCell>
              <TableCell>
                <Chip label="001" size="small" color="primary" variant="outlined" />
              </TableCell>
              <TableCell>
                <Box display="flex" alignItems="center" gap={1}>
                  <Avatar src="/avatars/yamada.jpg" sx={{ width: 32, height: 32 }}>山</Avatar>
                  山田太郎
                </Box>
              </TableCell>
              <TableCell>
                <Chip label="全権管理者" size="small" color="error" />
              </TableCell>
              <TableCell>yamada@example.com</TableCell>
              <TableCell>090-1234-5678</TableCell>
              <TableCell>2025-07-01</TableCell>
              <TableCell>
                <IconButton size="small" color="primary">
                  <VisibilityIcon />
                </IconButton>
                <IconButton size="small" color="inherit">
                  <EditIcon />
                </IconButton>
              </TableCell>
            </TableRow>
            
            <TableRow hover style="cursor: pointer;">
              <TableCell padding="checkbox">
                <Checkbox />
              </TableCell>
              <TableCell>
                <Chip label="002" size="small" color="primary" variant="outlined" />
              </TableCell>
              <TableCell>
                <Box display="flex" alignItems="center" gap={1}>
                  <Avatar src="/avatars/tanaka.jpg" sx={{ width: 32, height: 32 }}>田</Avatar>
                  田中花子
                </Box>
              </TableCell>
              <TableCell>
                <Chip label="一般管理者" size="small" color="warning" />
              </TableCell>
              <TableCell>tanaka@example.com</TableCell>
              <TableCell>090-5678-1234</TableCell>
              <TableCell>2025-07-02</TableCell>
              <TableCell>
                <IconButton size="small" color="primary">
                  <VisibilityIcon />
                </IconButton>
                <IconButton size="small" color="inherit">
                  <EditIcon />
                </IconButton>
              </TableCell>
            </TableRow>
            
            <TableRow hover style="cursor: pointer;">
              <TableCell padding="checkbox">
                <Checkbox />
              </TableCell>
              <TableCell>
                <Chip label="003" size="small" color="primary" variant="outlined" />
              </TableCell>
              <TableCell>
                <Box display="flex" alignItems="center" gap={1}>
                  <Avatar sx={{ width: 32, height: 32, bgcolor: '#1976d2' }}>佐</Avatar>
                  佐藤次郎
                </Box>
              </TableCell>
              <TableCell>
                <Chip label="スタッフ" size="small" color="info" />
              </TableCell>
              <TableCell>sato@example.com</TableCell>
              <TableCell>090-9876-5432</TableCell>
              <TableCell>2025-07-03</TableCell>
              <TableCell>
                <IconButton size="small" color="primary">
                  <VisibilityIcon />
                </IconButton>
                <IconButton size="small" color="inherit">
                  <EditIcon />
                </IconButton>
              </TableCell>
            </TableRow>
          </TableBody>
        </Table>
      </TableContainer>
      
      <!-- Pagination -->
      <Box display="flex" justifyContent="space-between" alignItems="center" p={2}>
        <Typography variant="body2" color="text.secondary">
          1-20 of 150 items
        </Typography>
        <Pagination count={8} page={1} color="primary" />
      </Box>
    </Paper>
  </Container>
</div>
```

### コンポーネント詳細

**Header**
- AppBar (Material-UI)
- ロゴ＋アプリケーション名 (左)
- ユーザー情報・ログアウト (右)
- 背景色: 青の濃淡グラデーション (#1565c0 - #1976d2)
- ログアウト時: 確認ダイアログ表示

**検索・フィルターエリア**
- Paper (Material-UI)
- 検索フィールド: TextField
  - placeholder: "氏名、メールアドレス、電話番号で検索"
- フィルター: Select, Checkbox
- ソート: Select
- アクションボタン: Button群

**スタッフ一覧テーブル**
- DataGrid (Material-UI)
- チェックボックス: 複数選択対応
- ソート機能: 各列でソート可能
- 行選択: 単一選択時は詳細画面へ遷移

**ページネーション**
- Pagination (Material-UI)
- 1ページ20件表示
- 件数表示: "1-20 of 150 items"

---

## 3. スタッフ詳細・メンテナンス画面

### レイアウト概要
```html
<!-- Material-UI Staff Detail Page Mockup -->
<div style="min-height: 100vh; background: #f3f7ff;">
  
  <!-- Header -->
  <AppBar position="static" style="background: linear-gradient(90deg, #1565c0 0%, #1976d2 100%);">
    <Toolbar>
      <Box display="flex" alignItems="center" flexGrow={1}>
        <Avatar src="/logo.svg" sx={{ mr: 2, width: 32, height: 32 }} />
        <Typography variant="h6" color="white" fontWeight="bold">
          スタッフ管理システム
        </Typography>
      </Box>
      <Box display="flex" alignItems="center" gap={2}>
        <Chip 
          avatar={<Avatar>山</Avatar>}
          label="山田太郎 (管理者)"
          variant="outlined"
          style="color: white; borderColor: white;"
        />
        <Button color="inherit" startIcon={<LogoutIcon />}>
          ログアウト
        </Button>
      </Box>
    </Toolbar>
  </AppBar>

  <!-- Navigation Bar -->
  <Paper elevation={1} style="padding: 16px;">
    <Box display="flex" justifyContent="space-between" alignItems="center">
      <Box display="flex" alignItems="center" gap={2}>
        <Button startIcon={<ArrowBackIcon />} variant="outlined">
          一覧に戻る
        </Button>
        <Typography variant="h5" color="#1565c0" fontWeight="bold">
          スタッフ詳細情報
        </Typography>
      </Box>
      <Box display="flex" gap={1}>
        <Button variant="contained" startIcon={<EditIcon />} style="background: #1976d2;">
          編集
        </Button>
        <Button variant="outlined" color="error" startIcon={<DeleteIcon />}>
          削除
        </Button>
      </Box>
    </Box>
  </Paper>

  <!-- Main Content -->
  <Container maxWidth="lg" style="padding: 24px;">
    
    <!-- Basic Information -->
    <Paper elevation={2} style="padding: 32px; margin-bottom: 24px; border-radius: 12px;">
      <Typography variant="h6" gutterBottom color="#1565c0" fontWeight="bold" style="border-bottom: 2px solid #e3f2fd; padding-bottom: 8px;">
        基本情報
      </Typography>
      
      <Grid container spacing={4} style="margin-top: 16px;">
        <Grid item xs={12} md={3}>
          <Box textAlign="center">
            <Avatar 
              src="/avatars/yamada.jpg" 
              sx={{ width: 120, height: 120, mx: 'auto', mb: 2, border: '4px solid #1976d2' }}
            >
              山
            </Avatar>
            <Button 
              variant="outlined" 
              size="small" 
              startIcon={<PhotoCameraIcon />}
              sx={{ margin: '0 auto', display: 'block', width: 'fit-content' }}
            >
              写真変更
            </Button>
          </Box>
        </Grid>
        <Grid item xs={12} md={9}>
          <Grid container spacing={3}>
            <Grid item xs={12} sm={6}>
              <Box>
                <Typography variant="subtitle2" color="text.secondary" gutterBottom>
                  ユーザーコード
                </Typography>
                <Chip label="user001" variant="outlined" color="primary" />
              </Box>
            </Grid>
            <Grid item xs={12} sm={6}>
              <Box>
                <Typography variant="subtitle2" color="text.secondary" gutterBottom>
                  氏名
                </Typography>
                <Typography variant="h6" fontWeight="bold">
                  山田太郎
                </Typography>
              </Box>
            </Grid>
            <Grid item xs={12} sm={6}>
              <Box>
                <Typography variant="subtitle2" color="text.secondary" gutterBottom>
                  メールアドレス
                </Typography>
                <Typography variant="body1">
                  yamada@example.com
                </Typography>
              </Box>
            </Grid>
            <Grid item xs={12} sm={6}>
              <Box>
                <Typography variant="subtitle2" color="text.secondary" gutterBottom>
                  役職
                </Typography>
                <Chip label="全権管理者" color="error" />
              </Box>
            </Grid>
            <Grid item xs={12} sm={6}>
              <Box>
                <Typography variant="subtitle2" color="text.secondary" gutterBottom>
                  電話番号
                </Typography>
                <Box display="flex" alignItems="center" gap={1}>
                  <PhoneIcon color="action" />
                  <Typography variant="body1">03-1234-5678</Typography>
                </Box>
              </Box>
            </Grid>
            <Grid item xs={12} sm={6}>
              <Box>
                <Typography variant="subtitle2" color="text.secondary" gutterBottom>
                  携帯電話
                </Typography>
                <Box display="flex" alignItems="center" gap={1}>
                  <PhoneAndroidIcon color="action" />
                  <Typography variant="body1">090-1234-5678</Typography>
                </Box>
              </Box>
            </Grid>
          </Grid>
        </Grid>
      </Grid>
    </Paper>

    <!-- Operation History -->
    <Paper elevation={2} style="padding: 24px; border-radius: 12px;">
      <Typography variant="h6" gutterBottom color="#1565c0" fontWeight="bold" style="border-bottom: 2px solid #e3f2fd; padding-bottom: 8px;">
        操作履歴
      </Typography>
      
      <TableContainer style="margin-top: 16px;">
        <Table>
          <TableHead style="background: #f5f5f5;">
            <TableRow>
              <TableCell><b>日時</b></TableCell>
              <TableCell><b>操作者</b></TableCell>
              <TableCell><b>操作内容</b></TableCell>
              <TableCell><b>変更内容</b></TableCell>
            </TableRow>
          </TableHead>
          <TableBody>
            <TableRow>
              <TableCell>
                <Box display="flex" alignItems="center" gap={1}>
                  <ScheduleIcon color="action" />
                  2025-07-17 14:30
                </Box>
              </TableCell>
              <TableCell>
                <Chip avatar={<Avatar>管</Avatar>} label="admin" size="small" />
              </TableCell>
              <TableCell>
                <Chip label="新規登録" color="success" size="small" />
              </TableCell>
              <TableCell>スタッフ情報を新規作成</TableCell>
            </TableRow>
            <TableRow>
              <TableCell>
                <Box display="flex" alignItems="center" gap={1}>
                  <ScheduleIcon color="action" />
                  2025-07-16 09:15
                </Box>
              </TableCell>
              <TableCell>
                <Chip avatar={<Avatar>管</Avatar>} label="manager" size="small" />
              </TableCell>
              <TableCell>
                <Chip label="情報更新" color="info" size="small" />
              </TableCell>
              <TableCell>電話番号を変更</TableCell>
            </TableRow>
            <TableRow>
              <TableCell>
                <Box display="flex" alignItems="center" gap={1}>
                  <ScheduleIcon color="action" />
                  2025-07-15 16:45
                </Box>
              </TableCell>
              <TableCell>
                <Chip avatar={<Avatar>管</Avatar>} label="admin" size="small" />
              </TableCell>
              <TableCell>
                <Chip label="権限変更" color="warning" size="small" />
              </TableCell>
              <TableCell>役職を「スタッフ」から「全権管理者」に変更</TableCell>
            </TableRow>
          </TableBody>
        </Table>
      </TableContainer>
      
      <Box display="flex" justifyContent="center" mt={2}>
        <Pagination count={3} page={1} color="primary" size="small" />
      </Box>
    </Paper>
  </Container>
</div>
```


### コンポーネント詳細

**アクション バー**
- Button群: 戻る、編集、削除
- 権限に応じて表示制御
- 削除ボタン: 確認ダイアログ表示
- 編集ボタン: 編集モードへの切り替え確認

**基本情報エリア**
- Grid (Material-UI)
- 顔写真: Avatar または ImageList
- 情報表示: Typography, Chip

**セキュリティ情報エリア**
- ImageList (Material-UI)
- 画像表示: 3x1 グリッド
- 画像クリック: 拡大表示 (Dialog)

**操作履歴**
- Table (Material-UI)
- ページネーション対応
- 日時順ソート

---

## 4. メール送信画面

### レイアウト概要
```html
<!-- Material-UI Mail Send Page Mockup -->
<div style="min-height: 100vh; background: #f3f7ff;">
  
  <!-- Header -->
  <AppBar position="static" style="background: linear-gradient(90deg, #1565c0 0%, #1976d2 100%);">
    <Toolbar>
      <Box display="flex" alignItems="center" flexGrow={1}>
        <Avatar src="/logo.svg" sx={{ mr: 2, width: 32, height: 32 }} />
        <Typography variant="h6" color="white" fontWeight="bold">
          スタッフ管理システム
        </Typography>
      </Box>
      <Box display="flex" alignItems="center" gap={2}>
        <Chip 
          avatar={<Avatar>山</Avatar>}
          label="山田太郎 (管理者)"
          variant="outlined"
          style="color: white; borderColor: white;"
        />
        <Button color="inherit" startIcon={<LogoutIcon />}>
          ログアウト
        </Button>
      </Box>
    </Toolbar>
  </AppBar>

  <!-- Navigation Bar -->
  <Paper elevation={1} style="padding: 16px;">
    <Box display="flex" justifyContent="space-between" alignItems="center">
      <Box display="flex" alignItems="center" gap={2}>
        <Button startIcon={<ArrowBackIcon />} variant="outlined">
          一覧に戻る
        </Button>
        <Typography variant="h5" color="#1565c0" fontWeight="bold">
          メール送信
        </Typography>
      </Box>
      <Box display="flex" gap={1}>
        <Button variant="contained" startIcon={<SendIcon />} style="background: #1976d2;">
          送信
        </Button>
        <Button variant="outlined" startIcon={<SaveIcon />}>
          下書き保存
        </Button>
      </Box>
    </Box>
  </Paper>

  <!-- Main Content -->
  <Container maxWidth="lg" style="padding: 24px;">
    
    <!-- Recipients Section -->
    <Paper elevation={2} style="padding: 24px; margin-bottom: 24px; border-radius: 12px;">
      <Typography variant="h6" gutterBottom color="#1565c0" fontWeight="bold" style="border-bottom: 2px solid #e3f2fd; padding-bottom: 8px;">
        送信対象
      </Typography>
      
      <Box mt={2}>
        <Box display="flex" alignItems="center" gap={2} mb={2}>
          <Chip 
            label="選択済みスタッフ: 15名" 
            color="primary" 
            icon={<PeopleIcon />}
            style="font-size: 16px; padding: 4px;"
          />
          <Button variant="outlined" size="small" startIcon={<VisibilityIcon />}>
            対象者一覧を表示
          </Button>
        </Box>
        
        <!-- Selected Recipients Preview -->
        <Box display="flex" flexWrap="wrap" gap={1}>
          <Chip 
            avatar={<Avatar>山</Avatar>}
            label="山田太郎" 
            size="small" 
            onDelete={() => {}}
            deleteIcon={<CancelIcon />}
          />
          <Chip 
            avatar={<Avatar>田</Avatar>}
            label="田中花子" 
            size="small" 
            onDelete={() => {}}
            deleteIcon={<CancelIcon />}
          />
          <Chip 
            avatar={<Avatar>佐</Avatar>}
            label="佐藤次郎" 
            size="small" 
            onDelete={() => {}}
            deleteIcon={<CancelIcon />}
          />
          <Chip 
            label="他12名..." 
            size="small" 
            variant="outlined"
            clickable
          />
        </Box>
      </Box>
    </Paper>

    <!-- Mail Compose Section -->
    <Paper elevation={2} style="padding: 24px; margin-bottom: 24px; border-radius: 12px;">
      <Typography variant="h6" gutterBottom color="#1565c0" fontWeight="bold" style="border-bottom: 2px solid #e3f2fd; padding-bottom: 8px;">
        メール作成
      </Typography>
      
      <Box mt={3}>
        <!-- Subject Field -->
        <TextField
          fullWidth
          label="件名"
          placeholder="件名を入力してください"
          variant="outlined"
          margin="normal"
          InputProps={{
            startAdornment: <SubjectIcon color="action" style="margin-right: 8px;" />
          }}
        />
        
        <!-- Body Field -->
        <TextField
          fullWidth
          label="本文"
          placeholder="メール本文を入力してください"
          variant="outlined"
          multiline
          rows={12}
          margin="normal"
          style="margin-top: 16px;"
        />
        
        <!-- Attachments Section -->
        <Box mt={3}>
          <Typography variant="subtitle1" gutterBottom>
            添付ファイル
          </Typography>
          
          <!-- ドラッグアンドドロップゾーン -->
          <Box 
            border="2px dashed #1976d2"
            borderRadius="12px"
            padding="24px"
            textAlign="center"
            bgcolor="#f3f7ff"
            mb={2}
            sx={{ 
              cursor: 'pointer',
              transition: 'all 0.3s ease',
              '&:hover': {
                bgcolor: '#e3f2fd',
                borderColor: '#1565c0',
                transform: 'scale(1.02)'
              }
            }}
          >
            <CloudUploadIcon sx={{ fontSize: 48, color: '#1976d2', mb: 1 }} />
            <Typography variant="h6" color="#1976d2" fontWeight="500">
              ファイルをドラッグ＆ドロップ
            </Typography>
            <Typography variant="body2" color="text.secondary">
              または下のボタンでファイルを選択
            </Typography>
          </Box>
          
          <Box display="flex" alignItems="center" gap={2}>
            <Button
              variant="outlined"
              startIcon={<AttachFileIcon />}
              component="label"
            >
              ファイル選択
              <input type="file" hidden multiple accept=".pdf,.jpg,.jpeg,.png,.gif" />
            </Button>
            <Typography variant="body2" color="text.secondary">
              最大10MB、PDF・画像ファイルに対応
            </Typography>
          </Box>
          
          <!-- Attached Files List -->
          <Box mt={2}>
            <Chip 
              label="重要なお知らせ.pdf (2.3MB)" 
              onDelete={() => {}}
              deleteIcon={<CancelIcon />}
              icon={<PictureAsPdfIcon />}
              style="margin-right: 8px; margin-bottom: 8px;"
            />
            <Chip 
              label="画像.jpg (1.5MB)" 
              onDelete={() => {}}
              deleteIcon={<CancelIcon />}
              icon={<ImageIcon />}
              style="margin-right: 8px; margin-bottom: 8px;"
            />
          </Box>
        </Box>
      </Box>
    </Paper>

    <!-- Preview Section -->
    <Paper elevation={2} style="padding: 24px; border-radius: 12px;">
      <Typography variant="h6" gutterBottom color="#1565c0" fontWeight="bold" style="border-bottom: 2px solid #e3f2fd; padding-bottom: 8px;">
        プレビュー
      </Typography>
      
      <Box mt={2} p={2} style="background: #fafafa; border-radius: 8px; border: 1px solid #e0e0e0;">
        <Box mb={2}>
          <Typography variant="body2" color="text.secondary">件名:</Typography>
          <Typography variant="h6" style="margin-top: 4px;">
            【重要】システムメンテナンスのお知らせ
          </Typography>
        </Box>
        
        <Divider style="margin: 16px 0;" />
        
        <Box>
          <Typography variant="body2" color="text.secondary" gutterBottom>本文:</Typography>
          <Typography variant="body1" style="white-space: pre-line; line-height: 1.6;">
            いつもお世話になっております。{'\n\n'}
            下記の通り、システムメンテナンスを実施いたします。{'\n'}
            メンテナンス中はシステムをご利用いただけませんので、{'\n'}
            ご理解とご協力をお願いいたします。{'\n\n'}
            【メンテナンス日時】{'\n'}
            2025年7月20日（日）午前2:00〜午前6:00{'\n\n'}
            ご不明な点がございましたら、お気軽にお問い合わせください。
          </Typography>
        </Box>
        
        <Box mt={2}>
          <Typography variant="body2" color="text.secondary">
            添付ファイル: 2件
          </Typography>
        </Box>
      </Box>
    </Paper>
  </Container>
</div>
```

### コンポーネント詳細

**送信対象エリア**
- Paper (Material-UI)
- Chip: 選択済み件数表示
- Button: 対象者一覧展開

**メール作成エリア**
- TextField: 件名入力
  - placeholder: "件名を入力してください"
- TextField: 本文入力 (multiline)
  - placeholder: "メール本文を入力してください"
- **ドラッグアンドドロップゾーン**: 視覚的なファイルドロップエリア
  - CloudUploadIcon: 48pxサイズのアップロードアイコン
  - ドラッグオーバー時の視覚フィードバック（背景色・スケール変更）
  - クリックでファイル選択ダイアログを開く
- Button: ファイル選択（従来の方法）
- **ファイル検証機能**:
  - 対応形式: PDF、JPG、PNG、GIF
  - 最大サイズ: 10MB
  - 重複チェック: 同名ファイルの重複防止
  - リアルタイムエラー表示
- FileList: 添付ファイル一覧
  - ファイルタイプ別アイコン（PDF: PictureAsPdfIcon、画像: ImageIcon）
  - ファイルサイズ表示
  - 削除ボタン付き
- **トースト通知**: 右上スライドイン式メッセージ
  - 成功: 緑背景
  - エラー: 赤背景
  - 情報: 青背景
- 送信ボタン: 送信前確認ダイアログ表示

**プレビューエリア**
- Paper (Material-UI)
- 送信前確認用
- リアルタイム更新

---

## 5. 新規登録画面

### レイアウト概要
```html
<!-- Material-UI Staff Registration Page Mockup -->
<div style="min-height: 100vh; background: #f3f7ff;">
  
  <!-- Header -->
  <AppBar position="static" style="background: linear-gradient(90deg, #1565c0 0%, #1976d2 100%);">
    <Toolbar>
      <Box display="flex" alignItems="center" flexGrow={1}>
        <Avatar src="/logo.svg" sx={{ mr: 2, width: 32, height: 32 }} />
        <Typography variant="h6" color="white" fontWeight="bold">
          スタッフ管理システム
        </Typography>
      </Box>
      <Box display="flex" alignItems="center" gap={2}>
        <Chip 
          avatar={<Avatar>山</Avatar>}
          label="山田太郎 (管理者)"
          variant="outlined"
          style="color: white; borderColor: white;"
        />
        <Button color="inherit" startIcon={<LogoutIcon />}>
          ログアウト
        </Button>
      </Box>
    </Toolbar>
  </AppBar>

  <!-- Navigation Bar -->
  <Paper elevation={1} style="padding: 16px;">
    <Box display="flex" justifyContent="space-between" alignItems="center">
      <Box display="flex" alignItems="center" gap={2}>
        <Button startIcon={<ArrowBackIcon />} variant="outlined">
          一覧に戻る
        </Button>
        <Typography variant="h5" color="#1565c0" fontWeight="bold">
          新規スタッフ登録
        </Typography>
      </Box>
      <Box display="flex" gap={1}>
        <Button variant="contained" startIcon={<SaveIcon />} style="background: #1976d2;">
          登録
        </Button>
        <Button variant="outlined" startIcon={<CancelIcon />}>
          キャンセル
        </Button>
      </Box>
    </Box>
  </Paper>

  <!-- Main Content -->
  <Container maxWidth="lg" style="padding: 24px;">
    
    <!-- Basic Information Section -->
    <Paper elevation={2} style="padding: 32px; margin-bottom: 24px; border-radius: 12px;">
      <Typography variant="h6" gutterBottom color="#1565c0" fontWeight="bold" style="border-bottom: 2px solid #e3f2fd; padding-bottom: 8px;">
        基本情報
      </Typography>
      
      <Grid container spacing={4} style="margin-top: 16px;">
        <Grid item xs={12} md={3}>
          <Box textAlign="center">
            <Avatar 
              sx={{ 
                width: 120, 
                height: 120, 
                mx: 'auto', 
                mb: 2, 
                border: '4px dashed #1976d2',
                backgroundColor: '#f3f7ff'
              }}
            >
              <PhotoCameraIcon sx={{ fontSize: 40, color: '#1976d2' }} />
            </Avatar>
            <Button 
              variant="outlined" 
              component="label"
              startIcon={<UploadIcon />}
              sx={{ 
                maxWidth: '180px',
                whiteSpace: 'nowrap',
                display: 'inline-flex'
              }}
            >
              顔写真アップロード
              <input type="file" hidden accept="image/*" />
            </Button>
            <Typography variant="caption" color="text.secondary" display="block" mt={1}>
              JPG、PNG（最大2MB）
            </Typography>
          </Box>
        </Grid>
        <Grid item xs={12} md={9}>
          <Grid container spacing={3}>
            <Grid item xs={12} sm={6}>
              <TextField
                fullWidth
                label="ユーザーコード"
                placeholder="例：user001"
                variant="outlined"
                required
                InputProps={{
                  startAdornment: <PersonIcon color="action" style="margin-right: 8px;" />
                }}
                helperText="半角英数字、20文字以内"
              />
            </Grid>
            <Grid item xs={12} sm={6}>
              <TextField
                fullWidth
                label="氏名"
                placeholder="山田太郎"
                variant="outlined"
                required
                InputProps={{
                  startAdornment: <AccountCircleIcon color="action" style="margin-right: 8px;" />
                }}
                helperText="50文字以内"
              />
            </Grid>
            <Grid item xs={12} sm={6}>
              <TextField
                fullWidth
                label="メールアドレス"
                placeholder="taro.yamada@example.com"
                variant="outlined"
                type="email"
                required
                InputProps={{
                  startAdornment: <EmailIcon color="action" style="margin-right: 8px;" />
                }}
                helperText="ログインID兼用"
              />
            </Grid>
            <Grid item xs={12} sm={6}>
              <TextField
                fullWidth
                label="パスワード"
                placeholder="英数字を含む8文字以上"
                variant="outlined"
                type="password"
                required
                InputProps={{
                  startAdornment: <LockIcon color="action" style="margin-right: 8px;" />
                }}
                helperText="英数字を含む8文字以上"
              />
            </Grid>
            <Grid item xs={12} sm={6}>
              <FormControl fullWidth required>
                <InputLabel>役職</InputLabel>
                <Select 
                  label="役職"
                  startAdornment={<WorkIcon color="action" style="margin-right: 8px;" />}
                >
                  <MenuItem value="全権管理者">
                    <Chip label="全権管理者" color="error" size="small" />
                  </MenuItem>
                  <MenuItem value="一般管理者">
                    <Chip label="一般管理者" color="warning" size="small" />
                  </MenuItem>
                  <MenuItem value="スタッフ">
                    <Chip label="スタッフ" color="info" size="small" />
                  </MenuItem>
                </Select>
                <FormHelperText>システム内での権限レベル</FormHelperText>
              </FormControl>
            </Grid>
            <Grid item xs={12} sm={6}>
              <TextField
                fullWidth
                label="電話番号"
                placeholder="03-1234-5678"
                variant="outlined"
                InputProps={{
                  startAdornment: <PhoneIcon color="action" style="margin-right: 8px;" />
                }}
                helperText="ハイフン有無問わず"
              />
            </Grid>
            <Grid item xs={12} sm={6}>
              <TextField
                fullWidth
                label="携帯電話"
                placeholder="090-1234-5678"
                variant="outlined"
                InputProps={{
                  startAdornment: <PhoneAndroidIcon color="action" style="margin-right: 8px;" />
                }}
                helperText="緊急連絡先"
              />
            </Grid>
          </Grid>
        </Grid>
      </Grid>
    </Paper>

    <!-- Validation Status Section -->
    <Paper elevation={2} style="padding: 24px; border-radius: 12px;">
      <Typography variant="h6" gutterBottom color="#1565c0" fontWeight="bold" style="border-bottom: 2px solid #e3f2fd; padding-bottom: 8px;">
        入力確認
      </Typography>
      
      <Box mt={2}>
        <!-- Required Fields Status -->
        <Typography variant="subtitle1" gutterBottom>
          必須項目チェック
        </Typography>
        <Grid container spacing={2}>
          <Grid item xs={12} sm={6} md={4}>
            <Box display="flex" alignItems="center" gap={1}>
              <CheckCircleIcon color="success" />
              <Typography variant="body2">ユーザーコード</Typography>
            </Box>
          </Grid>
          <Grid item xs={12} sm={6} md={4}>
            <Box display="flex" alignItems="center" gap={1}>
              <CheckCircleIcon color="success" />
              <Typography variant="body2">氏名</Typography>
            </Box>
          </Grid>
          <Grid item xs={12} sm={6} md={4}>
            <Box display="flex" alignItems="center" gap={1}>
              <CheckCircleIcon color="success" />
              <Typography variant="body2">メールアドレス</Typography>
            </Box>
          </Grid>
          <Grid item xs={12} sm={6} md={4}>
            <Box display="flex" alignItems="center" gap={1}>
              <ErrorIcon color="error" />
              <Typography variant="body2" color="error">パスワード</Typography>
            </Box>
          </Grid>
          <Grid item xs={12} sm={6} md={4}>
            <Box display="flex" alignItems="center" gap={1}>
              <ErrorIcon color="error" />
              <Typography variant="body2" color="error">役職</Typography>
            </Box>
          </Grid>
        </Grid>
        
        <!-- Validation Errors -->
        <Box mt={3}>
          <Alert severity="warning" style="margin-bottom: 8px;">
            <AlertTitle>入力エラーがあります</AlertTitle>
            以下の項目を確認してください
          </Alert>
          <Alert severity="error" style="margin-bottom: 8px;">
            パスワードは英数字を含む8文字以上で入力してください
          </Alert>
          <Alert severity="error" style="margin-bottom: 8px;">
            役職を選択してください
          </Alert>
        </Box>
        
        <!-- Registration Button -->
        <Box mt={3} textAlign="center">
          <Button 
            variant="contained" 
            size="large" 
            disabled={hasErrors}
            startIcon={<SaveIcon />}
            style="background: #1976d2; padding: 12px 48px;"
          >
            スタッフ情報を登録する
          </Button>
        </Box>
      </Box>
    </Paper>
  </Container>
</div>
```

### コンポーネント詳細

**基本情報エリア**
- Grid (Material-UI)
- TextField: 各入力フィールド
  - ユーザーコード: placeholder "例：user001"
  - 氏名: placeholder "山田太郎"
  - メール: placeholder "taro.yamada@example.com"
  - パスワード: placeholder "英数字を含む8文字以上"
  - 電話番号: placeholder "03-1234-5678"
  - 携帯電話: placeholder "090-1234-5678"
- Select: 役職選択
  - placeholder: "役職を選択してください"
- 必須項目表示: * マーク
- 登録ボタン: 入力内容確認ダイアログ表示
- キャンセルボタン: 入力破棄確認ダイアログ表示

**セキュリティ情報エリア**
- Button: ファイル選択
- FileList: 選択済みファイル表示
- 画像プレビュー: 選択後に表示

**入力確認エリア**
- Alert (Material-UI)
- バリデーション結果表示（日本語）
- 必須項目チェック状況
- エラー項目の具体的な修正指示

---

## レスポンシブデザイン対応

### タブレット表示 (768px以下)
- サイドバー: 折りたたみ
- テーブル: 横スクロール
- フォーム: 1カラム表示

### スマートフォン表示 (375px以下)
- ナビゲーション: ハンバーガーメニュー
- テーブル: カード形式表示
- フォーム: フルスクリーン

## 使用するMaterial-UIコンポーネント

- **Navigation**: AppBar, Drawer, Tabs
- **Inputs**: TextField, Select, Checkbox, Button, FileInput
- **Data Display**: Table, DataGrid, List, Avatar, Chip
- **Feedback**: Alert, Dialog, Snackbar, CircularProgress, Backdrop
- **Surfaces**: Paper, Card, Accordion
- **Layout**: Grid, Stack, Box, Container
- **Media**: Avatar (ロゴ表示用)
- **Icons**: 
  - CloudUploadIcon (ドラッグアンドドロップゾーン)
  - PictureAsPdfIcon (PDFファイル表示)
  - ImageIcon (画像ファイル表示)
  - PhotoCameraIcon (写真変更)
  - UploadIcon (アップロード)
  - AttachFileIcon (ファイル添付)
  - CancelIcon (削除/キャンセル)

## カラーパレット・デザインシステム

**基本カラーパレット**
```
Primary: #1976d2 (Blue)
Primary Dark: #1565c0 (Dark Blue)
Primary Light: #42a5f5 (Light Blue)
Secondary: #dc004e (Pink)
Success: #2e7d32 (Green)
Warning: #ed6c02 (Orange)
Error: #d32f2f (Red)
Info: #0288d1 (Light Blue)
Background: #f5f5f5 (Light Gray)
Surface: #ffffff (White)
```

**Header・Body デザイン**
- **Header**: 青の濃淡グラデーション (#1565c0 → #1976d2)
- **Body**: 薄い青系背景 (#f3f7ff)
- **Card/Paper**: 白背景 (#ffffff) + 青系のボーダー (#e3f2fd)
- **Button**: 青系統で統一、hover時に濃い青に変化

## アラート・メッセージシステム

**フローティングメッセージボックス**
- **成功メッセージ**: Snackbar (緑) + スライドイン アニメーション
- **エラーメッセージ**: Snackbar (赤) + フェードイン アニメーション
- **警告メッセージ**: Snackbar (オレンジ) + バウンス アニメーション
- **情報メッセージ**: Snackbar (青) + スライドダウン アニメーション
- **セッション警告**: Snackbar (オレンジ) + セッション残り時間表示

**アニメーション仕様**
- 表示時間: 4秒間（自動消去）
- 位置: 画面右上
- 重複時: 縦に積み重ね表示
- 手動閉じる: ×ボタン付き

## 操作確認ダイアログ

**確認が必要な操作**
1. **ログアウト**: "ログアウトしますか？"
2. **データ削除**: "このスタッフ情報を削除しますか？"
3. **メール送信**: "メールを送信しますか？（対象: XX名）"
4. **データ登録**: "この内容で登録しますか？"
5. **入力破棄**: "入力内容を破棄しますか？"
6. **編集モード**: "編集モードに切り替えますか？"
7. **セッション延長**: "セッションが間もなく期限切れになります。延長しますか？"

**ダイアログ仕様**
- Modal Dialog (Material-UI)
- 背景: Backdrop (半透明黒)
- ボタン: キャンセル（グレー）、実行（青）
- アニメーション: フェードイン/アウト
- キーボード操作: ESCキーでキャンセル

## インタラクション・アニメーション

**ボタンインタラクション**
- hover: 色変化 + 軽い影
- click: リップル効果
- disabled: 透明度50%

**テーブル・リストインタラクション**
- 行hover: 薄い青背景
- 選択行: 青背景 + 白文字
- ソート: 矢印アニメーション

**フォームインタラクション**
- フォーカス: 青のアウトライン
- エラー: 赤のアウトライン + 振動効果
- 成功: 緑のアウトライン + チェックアイコン

**ドラッグアンドドロップインタラクション**
- ドロップゾーン hover: 背景色変化 (#f3f7ff → #e3f2fd)
- ドラッグオーバー: スケールアップ (scale(1.02)) + ボーダー色変化
- ドロップ成功: トースト通知 (緑背景、右上スライドイン)
- ドロップエラー: トースト通知 (赤背景、エラーメッセージ)
- アニメーション: CSS transition 0.3s ease

## バリデーション・エラーメッセージ日本語化

**基本バリデーションメッセージ**
```javascript
// 必須項目
required: "この項目は必須です"

// 文字数制限
minLength: "最低{min}文字以上で入力してください"
maxLength: "最大{max}文字以内で入力してください"

// パターンマッチング
pattern: "正しい形式で入力してください"

// メールアドレス
email: "正しいメールアドレスを入力してください"

// 数値
min: "{min}以上の数値を入力してください"
max: "{max}以下の数値を入力してください"

// カスタムバリデーション
userCodeDuplicate: "このユーザーコードは既に使用されています"
passwordWeak: "パスワードは英数字を含む8文字以上で入力してください"
```

**項目別バリデーションメッセージ**
- **ユーザーコード**: 
  - 必須: "ユーザーコードを入力してください"
  - 形式: "半角英数字で入力してください"
  - 重複: "このユーザーコードは既に使用されています"
  - placeholder: "例：user001"
- **氏名**: 
  - 必須: "氏名を入力してください"
  - 文字数: "氏名は50文字以内で入力してください"
  - placeholder: "山田太郎"
- **メールアドレス**: 
  - 必須: "メールアドレスを入力してください"
  - 形式: "正しいメールアドレスを入力してください"
  - 重複: "このメールアドレスは既に登録されています"
  - placeholder: "taro.yamada@example.com"
- **パスワード**: 
  - 必須: "パスワードを入力してください"
  - 強度: "パスワードは英数字を含む8文字以上で入力してください"
  - placeholder: "英数字を含む8文字以上"
- **電話番号**: 
  - 形式: "正しい電話番号を入力してください（例：03-1234-5678）"
  - placeholder: "03-1234-5678"
- **携帯電話**: 
  - 形式: "正しい携帯電話番号を入力してください（例：090-1234-5678）"
  - placeholder: "090-1234-5678"

**ファイルアップロードバリデーション**
- **ファイル形式**: "対応ファイル形式: JPG, PNG, PDF"
- **ファイルサイズ**: "ファイルサイズは{maxSize}MB以内にしてください"
- **必須ファイル**: "顔写真をアップロードしてください"

**リアルタイムバリデーション**
- 入力中: 文字数カウンター表示
- フォーカス離脱時: 即座にバリデーション実行
- 送信前: 全項目の最終バリデーション

**バリデーション表示仕様**
- 位置: 入力フィールド直下
- 色: エラー (#d32f2f)
- アイコン: 警告アイコン付き
- アニメーション: フェードイン表示

**ファイルアップロードバリデーション**
- **ファイル形式チェック**: PDF, JPG, PNG, GIF以外は拒否
- **ファイルサイズ制限**: 10MB超過時はエラー表示
- **重複ファイルチェック**: 同名ファイルの添付を防止
- **エラー表示方法**: Toast通知（3秒間表示）
- **許可通知**: 正常添付時もToast表示（2秒間）
- **リアルタイムバリデーション**: ドロップ時即座にチェック実行
- **視覚的フィードバック**: ドラッグ中の境界色変化でバリデーション状態を表示

**placeholder仕様**
- 色: グレー (#9e9e9e)
- フォントスタイル: 通常テキストより薄い
- フォーカス時: 自動的に非表示
- 入力開始時: 自動的に非表示
- 具体的な入力例を提示してユーザビリティを向上

## 認証・セッション管理システム

**Laravel Sanctum による認証**
- **トークン発行**: ログイン成功時にPersonal Access Tokenを発行
- **トークン有効期限**: 3時間（180分）
- **トークン格納**: ブラウザのlocalStorageに安全に保存
- **API通信**: 全てのAPIリクエストにBearerトークンを付与
- **CSRF保護**: Sanctumの標準CSRF保護機能を利用

**セッション管理**
- **自動延長**: アクティブ時に残り30分で自動延長（新しいトークン発行）
- **セッション監視**: 1分間隔でトークン有効期限をチェック
- **アクティビティ検出**: マウス操作、キーボード操作、API通信を検出
- **セッション切れ**: 期限切れ時に自動的に認証画面へリダイレクト
- **ブラウザ履歴制御**: 認証後は戻るボタンで認証画面に戻れないよう制御

**セッション切れ対応**
- **10分前警告**: "セッションが間もなく期限切れになります。延長しますか？"
- **5分前警告**: "セッションが5分後に期限切れになります"
- **自動ログアウト**: 期限切れ時に自動的に認証画面へリダイレクト
- **作業中データ保護**: 入力中のデータは一時保存してセッション復旧後に復元
- **ブラウザ履歴管理**: 認証後は history.replaceState() で戻る操作を制御

**セッション表示**
- **セッション状況**: 通常時は表示なし
- **警告時のみ**: セッション延長確認ダイアログ表示
- **期限切れ時**: 認証画面への自動リダイレクト

**セキュリティ対策**
- **トークン暗号化**: Sanctumの標準的な暗号化機能を利用
- **CSRF対策**: Sanctumの内蔵CSRF保護とCSRFトークンを併用
- **XSS対策**: トークンの適切な保存と送信（httpOnlyオプション併用）
- **ブラウザ閉じた時**: トークンを自動削除（sessionStorage利用）
- **履歴制御**: 認証後の戻るボタンで認証画面に戻れないよう制御
- **ページ更新制御**: 認証状態の確認と維持
- **トークン無効化**: ログアウト時にサーバーサイドでトークンを無効化

**Laravel Sanctum の利点**
- **Laravel標準**: Laravel公式の認証システム
- **SPA最適化**: Single Page Applicationに最適化された設計
- **セキュリティ**: 標準的なセキュリティ機能が組み込み済み
- **簡単実装**: 複雑な設定なしで安全な認証が可能
- **トークン管理**: サーバーサイドでのトークン管理・無効化が可能
- **CSRF保護**: 標準でCSRF攻撃に対する保護機能を提供
- **レート制限**: API呼び出し制限機能が組み込み済み

## ローディング・プログレス

**ローディング表示**
- 全画面: CircularProgress + Backdrop
- 部分的: LinearProgress (青)
- ボタン内: 小さなSpinner

**プログレス表示**
- ファイルアップロード: 進捗バー
- データ処理: ステップ式プログレス
- 検索結果: スケルトンローディング

## ロゴ仕様

**ロゴ配置**
- 位置: Header左端（アプリケーション名の左側）
- サイズ: 40x40px (デスクトップ) / 32x32px (モバイル)
- フォーマット: SVG推奨（PNG/JPEGも対応）
- 余白: 左右8px

**ロゴデザイン要件**
- 背景透過
- 単色またはシンプルな配色
- 視認性を考慮したコントラスト
- レスポンシブ対応（サイズ自動調整）

## カラーパレット

**廃止**: 以下の内容は上記「カラーパレット・デザインシステム」セクションに統合されました。

## アクセシビリティ考慮事項

- **キーボードナビゲーション**: Tab順序の最適化
- **スクリーンリーダー**: ARIA属性の適切な設定
- **コントラスト**: WCAG 2.1 AA準拠（青系統での十分なコントラスト確保）
- **フォーカス**: 視覚的インジケーターの提供（青のアウトライン）
- **カラーバリアフリー**: 色以外でも情報を判別可能
- **アニメーション**: 視覚的な負担を軽減する設定
