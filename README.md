 <!-- col5: 固定尺寸的 Set 按鈕，呼叫 VM 的命令，參數 = 這一列的 ConfigUIRaw -->
          <Button Grid.Column="4" Content="Set"
                  Width="60" Height="30" HorizontalAlignment="Center"
                  Command="{Binding DataContext.ApplyRowCommand,
                                    RelativeSource={RelativeSource AncestorType=ItemsControl}}"
                  CommandParameter="{Binding}"/>