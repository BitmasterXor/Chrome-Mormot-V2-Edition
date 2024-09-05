object Form1: TForm1
  Left = 0
  Top = 0
  Caption = 'Chromium Password Recovery'
  ClientHeight = 537
  ClientWidth = 727
  Color = clBtnFace
  Font.Charset = DEFAULT_CHARSET
  Font.Color = clWindowText
  Font.Height = -12
  Font.Name = 'Segoe UI'
  Font.Style = []
  Position = poScreenCenter
  DesignSize = (
    727
    537)
  TextHeight = 15
  object ListView1: TListView
    Left = 0
    Top = 0
    Width = 726
    Height = 483
    Anchors = [akLeft, akTop, akRight, akBottom]
    Columns = <
      item
        AutoSize = True
        Caption = 'URL'
      end
      item
        AutoSize = True
        Caption = 'User Name'
      end
      item
        AutoSize = True
        Caption = 'Password'
      end>
    TabOrder = 0
    ViewStyle = vsReport
  end
  object Button1: TButton
    Left = 0
    Top = 488
    Width = 726
    Height = 49
    Anchors = [akLeft, akRight, akBottom]
    Caption = 'Find And Decrypt Chrome Passwords!'
    TabOrder = 1
    OnClick = Button1Click
  end
  object ipwJSON1: TipwJSON
    Left = 32
    Top = 96
  end
  object AESGCM1: TAESGCM
    Version = '4.0.0.0'
    keyLength = kl256
    outputFormat = raw
    IVMode = userdefined
    IVLength = 12
    Unicode = noUni
    Left = 32
    Top = 152
  end
  object FDConnection: TFDConnection
    Left = 32
    Top = 208
  end
  object FDQuery: TFDQuery
    Connection = FDConnection
    Left = 32
    Top = 264
  end
  object FDGUIxWaitCursor: TFDGUIxWaitCursor
    Provider = 'Forms'
    Left = 32
    Top = 320
  end
  object OperatingSystemInfo1: TOperatingSystemInfo
    Host = '.'
    Active = False
    Left = 56
    Top = 40
  end
end
